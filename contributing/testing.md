# Testes

## Backend

| | E2E | Import de instalação |
| --- | --- | --- |
| Config | `vitest.e2e.config.ts` | `vitest.import.config.ts` |
| Setup | `test/setup.e2e.ts` (MongoDB por suíte, DB único `test_{uuid}`) | idem |
| Banco | MongoDB real | MongoDB real, com origem legada semeada |
| Workers | 1 (`maxWorkers: 1`) | 1 |
| Timeout | 60s | 60s |
| Pattern | `*.controller.spec.ts` | o próprio diretório da extensão |
| Runner | forks | forks |

A suíte **unitária** é `*.unit.spec.ts`, e só ela. `vitest.unit.config.ts` +
`pnpm test:unit` rodam esse padrão e **não sobem nada** — nem Mongo, nem
kernel. Caso que depende de request entra no `*.controller.spec.ts` da
fatia; cálculo puro (fatiamento, limite, inferência, de-para) entra num
`*.unit.spec.ts`.

```bash
cd backend
pnpm test             # e2e (alias de test:e2e)
pnpm test:unit        # unitário, sem infraestrutura
pnpm test:e2e         # e2e contra MongoDB real
pnpm test:e2e:postgres # mesma suíte e2e com TEST_DATA_DRIVER=postgres
pnpm test:import      # suíte de import de instalação legada
pnpm test:coverage    # coverage (V8)
```

`test:e2e:postgres` exige `DATA_POSTGRES_URL` definida em `.env.test` — sem
ela, a suíte e o `postgres-driver.controller.spec` são pulados em silêncio.

### Helpers de teste

`test/helpers/auth.helper.ts`:

- `createAuthenticatedUser(overrides?)` — semeia as 26 permissões pela mesma
  lista do seeder, materializa os 4 grupos de sistema, cria a conta MASTER,
  faz sign-in e devolve cookies + user.
- `cleanDatabase()` — deleta `User` e `UserGroup`.

### Rodando a suíte completa com relatório (roteiro para sessão isolada)

O repositório documenta um roteiro próprio (`RODAR-TESTES-BACKEND.md`) para
rodar todas as verificações automatizadas e **coletar os erros num
relatório**, sem corrigir nada. Pontos relevantes:

- Máquina de referência com 6 GB de RAM: etapas **sequenciais**, com teto de
  heap por comando (`tsc` 2500 MB, `eslint` 3500 MB, `vitest` 3500 MB) e uma
  trava de arquivo (`flock`) para não disputar memória com outra sessão.
- Containers de teste dedicados (`lcjs-test-mongo`, `lcjs-test-redis`,
  `lcjs-test-pg`), com `--ulimit nofile=64000:64000` no Mongo — sem isso ele
  morre com "Too many open files" no meio da suíte.
- `backend/.env.test` precisa ter `JWT_PUBLIC_KEY`, `JWT_PRIVATE_KEY`,
  `COOKIE_SECRET` (32+ caracteres), `STORAGE_URL_SIGNING_KEY`,
  `PASSWORDS_ENCRYPTION_KEY` e `DATA_POSTGRES_URL` descomentada.
- Em terminal não interativo, o Vitest só imprime no fim — um log parado não
  significa que travou; conferir com `pgrep -fa vitest` e o número de
  databases `test_*` no Mongo.

### Checagens estáticas (gate de cada tarefa)

```bash
cd backend
pnpm exec tsc --noEmit
pnpm run openapi:check
pnpm run di:check
pnpm run scope:check
pnpm run boot:check
pnpm lint
```

`pnpm check` inclui `openapi:verify`, que pode divergir do baseline por
motivos não relacionados à tarefa corrente — os gates de tarefa usam as
partes individuais acima.

## Frontend

```bash
cd frontend
pnpm test             # Vitest
```

A suíte cobre `src/**` e `extensions/**` (`*.test.ts`, `*.spec.ts`). Testes
que precisam de DOM declaram `// @vitest-environment jsdom` no topo do
arquivo. Não há suíte de componente (snapshot/RTL) no repositório — a camada
de componente/rota é conferida só por gate estático (lint + tsc + build).

### Roteiro completo (`RODAR-TESTES-FRONTEND.md`)

Sessão isolada, sem backend nem banco (as suítes do frontend são
independentes):

```bash
pnpm exec prettier --check .
pnpm exec eslint "**/*.{ts,tsx}" -f json -o <saida>.json
pnpm exec tsc --noEmit --pretty false
pnpm exec vitest run --reporter=default --reporter=json --outputFile.json=<saida>.json
VITE_API_BASE_URL=http://localhost:3000 pnpm exec vite build
```

No build de produção, vale procurar no log por `warning`, `(!)`, `Circular
dependency`, `Use of eval`, chunks acima de 1 MB e erros de plugin.

Uma etapa opcional de fumaça do SSR (`node .output/server/index.mjs`, com
`curl` nas rotas públicas) só roda se a API estiver no ar — um `500` em
qualquer rota pública aí é bloqueante.

## Regra de execução

Por convenção do repositório ([ADR 0003](../architecture/decisions.md)), o
agente que implementa uma correção **escreve** o teste de regressão, mas quem
**roda** a suíte completa é o mantenedor — o custo (MongoDB real, timeouts de
60s+) e o efeito colateral da execução ficam do lado de quem decide rodar.
