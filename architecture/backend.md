# Arquitetura — Backend

Plataforma construída com Fastify + TypeScript + MongoDB.

## Tech stack

| Tecnologia  | Versão                                    | Uso                                                |
| ------------ | ------------------------------------------ | ---------------------------------------------------- |
| Fastify      | 5.12.3                                      | HTTP framework                                        |
| TypeScript   | 6.0.3                                        | Linguagem                                             |
| MongoDB      | driver `mongodb` 7.6.0 via `@fastify/mongodb` 10 | Banco de dados, **sem ODM** — models são `defineModel` + store sobre `ScopedCollection` |
| PostgreSQL   | `pg` 8.23 via `@fastify/postgres` 6.1 (opcional) | Banco das **tabelas de dados**, quando o wizard o escolhe |
| Redis (ioredis) | 6.0.0                                    | Cache                                                 |
| Socket.IO    | 4.8.3                                        | WebSocket (chat)                                      |
| VineJS       | 4.4.0                                        | Validação                                             |
| JWT (RS256)  | —                                             | Autenticação                                          |
| AWS SDK v3   | 3.x                                           | Storage S3: objeto, multipart e URL pré-assinada      |
| Sharp        | 0.35.4                                       | Processamento de imagem                                |
| Nodemailer   | 10.0.0                                       | Email                                                  |
| isolated-vm  | 7.0.1                                        | Sandbox dos scripts de usuário (isolate com heap e realm próprios) |
| BullMQ       | 6.3.2                                        | Filas sobre o Redis                                    |
| Vitest       | 5.0.0                                        | Testes (unit + e2e)                                    |
| fastify-decorators | 3.16.1                                 | DI + Controller decorators                             |

## Arquitetura em camadas

```
Cliente HTTP → Middleware → Controller → Validator (VineJS)
                                 │
                                 ▼
                             Use Case ──► Store da entidade (ScopedCollection + entity())
                                 │              │
                                 ▼              ▼
                           Service Contract   MongoDB (DB_DATABASE — users, tables, fields, settings...)
                                 │
                                 ▼
                     RowStore(table) — montado em runtime
                                 │
                        ┌────────┴────────┐
                        ▼                 ▼
               MongoDB (DB_DATA_DATABASE)   PostgreSQL (DATA_POSTGRES_URL)
```

A aplicação tem **um `MongoClient`**, aberto pelo plugin `@fastify/mongodb`
como primeiro plugin do kernel, e **dois databases**: o de sistema (models
nativos) e o de dados (collections dinâmicas, um database por ambiente). Não
há ODM: cada entidade tem uma **definição** (`defineModel`, em
`application/model/`) e um **store** (`services/<x>/store/`) que expõe a
`ScopedCollection` do driver e a serialização para o domínio.

## Estrutura de diretórios

```
backend/
├── bin/server.ts                  # Entry point - inicia HTTP + Socket.IO
├── start/
│   ├── kernel.ts                  # Fastify kernel - plugins, CORS, JWT, Swagger
│   └── env.ts                     # Checagem de env vars a mão (síncrona)
├── config/
│   └── openapi.config.ts          # Metadados do documento OpenAPI
├── application/
│   ├── core/                      # Só tipo, valor e bootstrap
│   ├── middlewares/                # 9 middlewares, adaptador + contract + impl
│   ├── model/                      # Definições (defineModel) dos 23 models
│   ├── services/                   # Toda a lógica de comportamento
│   └── features/                  # 16 pastas de entidade, a superfície REST
│                                   # (188 controllers; extensões somam mais 15)
├── database/
│   ├── seeders/                    # Permissions, user groups, settings
│   └── migrations/                 # Migrações one-time (dual-connection)
├── docker-entry-point.sh           # Roda migrations, seeders e sync Postgres
├── extensions/                     # Pacotes de extensões (plugins/módulos/tools)
├── templates/email/                 # EJS templates
└── test/                           # Setup, helpers (auth)
```

## Responsabilidades por camada

### Controller (`*.controller.ts`)

Somente HTTP: parse request, chamar validator, delegar ao use-case, formatar
response. Recebe injeção de middleware via decorator `onRequest`. Não contém
lógica de negócio.

### Validator (`_shared.validator.ts`)

Validators VineJS, um por fatia, criados com `vine.create({...})`. O schema da
rota **não é escrito à mão**: `pnpm openapi:generate` lê o
`await XValidator.validate(request.body)` do controller e monta tudo em
`core/openapi/schema.generated.ts`.

### Use Case (`*.use-case.ts`)

Lógica de negócio pura. Retorna `Either<HTTPException, T>` (`Left` = erro,
`Right` = sucesso). Recebe services via **constructor injection** (`@Service`
resolve pelos tipos dos parâmetros). Não conhece HTTP.

```typescript
const result = await useCase.execute(input);
if (result.isLeft()) return this.http.sendError(response, result.value);
return response.status(200).send(result.value);
```

O `return` do reply não é estilo — é o que evita derrubar o processo: o reply
do Fastify é um thenable, e devolvê-lo garante que a promise do handler só
resolve depois do `send` já ter fechado.

### Acesso a dados — sem repositório

Use-case e service injetam o **store** da entidade (`XStoreContractService`) e
falam com a collection escopada dele: `this.users.collection().findOne(...)`.
O store não tem método de consulta por caso de uso — só `collection()`,
serialização, `hydrate()` e os atalhos genéricos (`read`, `list`, `create`).

Rows não têm definição declarada: `RowStore`
(`services/table/model-builder/`) monta a collection em runtime a partir do
`_schema` da tabela, no banco que a instalação escolheu.

### Service (`*-contract.service.ts` + implementação)

Regra reutilizada por 2+ use-cases, ou que precisa de dependência injetada.
Contract abstrato + `export default` da impl no arquivo irmão, registrado no DI
via `di-registry.core.ts`.

### Middleware

Nove, cada um um adaptador sobre contract + impl `@Service`:

- `authentication.middleware.ts` — lê o JWT só do cookie `accessToken`
- `permission.middleware.ts` — exige capacidade de área (`E_AREA_CAPABILITY`)
- `role.middleware.ts` — papel de sistema (MASTER/ADMINISTRATOR), pelo fecho de
  grupos
- `table-access.middleware.ts` — bindings por ação + perfil de membro + dono
- `workspace-target.middleware.ts` — cruza `:_id` da rota com o plano do
  header
- `extension-active.middleware.ts` — 404 quando a extensão está desligada
- `rate-limit.middleware.ts` — cota no Redis, headers `x-ratelimit-*`, 429
- `setup-bootstrap.middleware.ts` — etapas do Setup Wizard
- `ai-access.middleware.ts` — credencial de agente da rota MCP

### Model (`*.model.ts`)

`defineModel<T>({ name, collection, scoped, softDelete, fields, indexes })`,
tipado contra a entidade de domínio; timestamps automáticos. Soft delete via
`trashed` + `trashedAt`.

## Padrões de design

### Nada de função solta

Comportamento vive em **service injetável** (contract + impl), agrupado por
contexto. `application/core/` guarda apenas tipo, valor e bootstrap. Exceções
pontuais (validator de escopo, migration/seeder, middleware, `schedulerDiscovery`,
defaults de model) estão comentadas no próprio código.

### Either/Result pattern

Padrão único de retorno de use-case, descrito acima.

### Soft delete

Todas as entidades usam `trashed: boolean` + `trashedAt: Date | null`. Dados
nunca são hard-deleted, exceto via operações específicas (`hard-delete` no
menu, `empty-trash`).

### Schema dinâmico

Tabelas possuem `_schema` (Mixed) convertido em runtime num `RowStore`
(`ModelBuilderContractService.build(table)`). No Postgres o mesmo `_schema`
(mais `table.fields`) vira a forma relacional — coluna tipada por campo,
tabela filha por grupo, FK por relacionamento.

### Sandbox de script

Código de usuário (`beforeSave`, `afterSave`, `onLoad`) roda num isolate do
`isolated-vm` (heap de 128 MB, timeout de 5s). APIs disponíveis: `field`,
`context`, `email`, `users`, `notify`, `utils`, `console` — todas como ponte
serializada, sem referência do host atravessando a fronteira.

## Enums core principais

| Enum | Valores |
| --- | --- |
| `E_ROLE` | MASTER, ADMINISTRATOR, MANAGER, REGISTERED |
| `E_FIELD_TYPE` | TEXT_SHORT, TEXT_LONG, DROPDOWN, DATE, RELATIONSHIP, FILE, FIELD_GROUP, REACTION, EVALUATION, CATEGORY, USER, USER_GROUP, HTML_CONTENT + nativos |
| `E_TABLE_STYLE` | LIST, GALLERY, DOCUMENT, CARD, MOSAIC, KANBAN, FORUM, CALENDAR, GANTT |
| `E_TABLE_PERMISSION` | CREATE/UPDATE/REMOVE/VIEW para TABLE, FIELD, ROW (12 total) |
| `E_AREA_CAPABILITY` | 14 capacidades de área — ver [RBAC](../features/rbac.md) |
| `E_TABLE_PROFILE` | OWNER, ADMIN, EDITOR, CONTRIBUTOR, VIEWER |
| `E_JWT_TYPE` | ACCESS, REFRESH |

## Planos: instalação e ambientes

A instalação vira o ambiente padrão. Todo dado de domínio mora dentro de um
ambiente (`workspace`), e o primeiro deles é a própria instalação convertida.
`application/core/workspace-scope.core.ts` guarda o plano da execução corrente
num `AsyncLocalStorage`, aplicado a todo model `scoped: true`, **fail-closed**.
Três planos:

| Plano | Como entrar | Filtro aplicado |
| --- | --- | --- |
| `installation` | `enterInstallationScope()` | `{ workspace: null }` |
| `workspace` | `enterWorkspaceScope(ws)` | `{ workspace: <id> }` |
| `unfiltered` | `runUnfilteredAcrossWorkspaces(cb)` | nenhum — enxerga tudo |

Toda request abre um plano a partir do header `x-workspace` (sem ele, cai na
instalação). Contexto **ausente** fora de request **lança** — o que impede o
default-leak de dado entre tenants.

## Sistema de permissões (RBAC)

Veja [RBAC](../features/rbac.md) para o detalhe completo. Resumo: grupos
custom com hierarquia (`encompasses`), capacidades de área
(`E_AREA_CAPABILITY`) e bindings por ação (`table.permissions`,
`field.permissions`, `menu.visibility`). Os 4 roles legados (MASTER >
ADMINISTRATOR > MANAGER > REGISTERED) seguem existindo só para compat no JWT.

## Dependency Injection (DI)

`application/core/di-registry.core.ts` registra os bindings **dinamicamente**:
varre o filesystem e pareia cada `<base>-contract.service.ts` com
`<base>.service.ts` por convenção — não há lista manual. Roots varridos:
`application/services`, `application/middlewares`, `hooks` e `extensions/`.
Hoje são 206 pares contrato/impl.

Para adicionar nova dependência (zero edição no di-registry):

1. Crie `<base>-contract.service.ts` com a abstract class `<X>ContractService`
2. Crie `<base>.service.ts` com `@Service() export default class` da impl
3. Consuma via constructor injection, importando o Contract pelo caminho
   direto do módulo (nunca por barrel `index.ts` — o SWC elide o tipo em
   re-export barrel e a injeção vira `undefined` silenciosamente)

## Fluxo de inicialização

```
bin/server.ts:
1. kernel.ready() - inicializa Fastify com todos os plugins; o primeiro
   (@fastify/mongodb) abre o MongoClient
2. kernel.listen({ port: Env.PORT, host: '0.0.0.0' })
3. initChatSocket(httpServer, jwtDecode) - Socket.IO para chat
```

Em container Docker, `docker-entry-point.sh` roda **antes** do servidor:

1. Migrations (`scripts/migrations/*.sh`, ordem 01→68, idempotentes)
2. `pnpm seed` (idempotente)
3. `scripts/postgres/sync-data-schema.sh` (no-op no Mongo; sincroniza as
   tabelas de cada ambiente quando a instalação usa Postgres)
4. Inicia o servidor

Desligamento (SIGTERM/SIGINT): para os workers de fila, fecha o Socket.IO,
chama `kernel.close()` (espera requests em voo, roda os `onClose`: scheduler,
Mongo, Postgres) e fecha o Redis compartilhado.

## Formato de resposta

### Sucesso

```json
{ "data": [...], "meta": { "total": 100, "page": 1, "perPage": 10, "lastPage": 10, "firstPage": 1 } }
```

### Erro

```json
{ "message": "Not found", "status": 404, "code": "TABLE_NOT_FOUND", "errors": { "campo": "mensagem" } }
```

`status` é o código HTTP; `code` é o identificador estável que o cliente
compara para decidir o fluxo — **nunca** a `message`, que é texto de UI.

## Variáveis de ambiente

Conferidas em `start/env.ts` à mão, sem schema — o módulo é avaliado no
import, antes do kernel e do container de DI. O `.env` cobre só
**infraestrutura** (DB, JWT, cookies, CORS, Redis, MCP, workers).
**Configurações de domínio** (branding, locale, upload, paginação, logos, IA,
SMTP, storage driver/S3) vivem no documento `Setting` do MongoDB e são
editadas via UI `/settings` pelo usuário MASTER.

Principais grupos de variáveis: banco de dados (`DATABASE_URL`,
`DB_DATABASE`, `DB_DATA_DATABASE`, `DATA_POSTGRES_URL`), JWT & cookies
(`JWT_PUBLIC_KEY`, `JWT_PRIVATE_KEY`, `COOKIE_SECRET`,
`STORAGE_URL_SIGNING_KEY`, `PASSWORDS_ENCRYPTION_KEY`,
`SETUP_BOOTSTRAP_SECRET`), servidor (`PORT`, `APP_SERVER_URL`,
`APP_CLIENT_URL`), CORS (`ALLOWED_ORIGINS`) e Redis (`REDIS_URL`). Veja
[Instalação](../getting-started/installation.md) para a lista com defaults.

## Error handling

`HTTPException` estende Error, com ~40 métodos factory estáticos
(`BadRequest`, `Unauthorized`, `NotFound`, `Forbidden`, etc.). O global error
handler (`services/error-handler/`) captura, em ordem: `HTTPException` →
erro de validação VineJS (422) → `BSONError`/id malformado (400) → índice
único duplicado (409) → erro 4xx do próprio Fastify → fallback 500.

## Migrations

Migrações one-time (`database/migrations/NN-migrate-*.ts`, idempotentes via
marcadores no Setting singleton). Rodam automaticamente no boot Docker; em
dev local (`pnpm dev`) não rodam. Não existem `pnpm migrate:*` — cada migration
tem um wrapper `.sh` em `scripts/migrations/`.

## Extensões

Mecanismo build-time + ativação runtime para estender a plataforma sem mexer
no core. Veja [Extensões](../features/extensions.md).

## Build & Deploy

- **Build**: `tsc --noEmit && tsup` → `/build` (ESM, ES2024, sem bundle de
  node_modules)
- **Dev**: `@swc-node/register` para transpilação rápida
- 3 Dockerfiles: `Dockerfile-local` (dev), `Dockerfile-production` (porta
  3000, usuário non-root), `Dockerfile-coolify` (multi-stage)
