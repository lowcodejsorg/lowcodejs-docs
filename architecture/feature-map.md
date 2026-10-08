# Arquitetura — Mapa de Funcionalidades

Resumo navegável de `_doc-mapa-funcionalidades.md` (auditoria de paridade de
2026-09-13, backend). O documento original é a régua de tudo que o backend
faz — rotas × middleware × use-case × services × extensões × migrations.
241 rotas no total, nenhuma removida numa migração recente do driver de
banco; 156 arquivos de spec e2e, todos cobrindo as rotas, rodando nos dois
drivers de dados (Mongo e Postgres).

## Legenda de middleware

`Auth` (JWT obrigatório) · `Auth?` (opcional) · `Perm(X)` (capacidade de área)
· `TA(X)` (acesso à tabela por ação) · `Role[...]` (papel de sistema) · `RL`
(rate limit) · `SB` (bootstrap do wizard) · `WT` (alvo de ambiente) ·
`AiAccess` (credencial MCP) · `ExtActive` (extensão ligada).

## Por área funcional

### Raiz e instância

`GET /`, `GET /health-check`, `GET /version`,
`GET /.well-known/lowcodejs.json` — sem auth.

### Autenticação (`/authentication`)

Sign-in/sign-up/sign-out, switch-account, accounts, magic-link,
refresh-token, recovery (request-code/validate-code/update-password). Todas
as rotas anônimas levam rate limit (`RL`).

### Usuários e grupos (`/users`, `/profile`, `/user-group`, `/permissions`)

CRUD completo + paginação + export CSV + trash/restore em lote, todas atrás
de `Auth` + `Perm(MANAGE_USERS)`/`Perm(MANAGE_USER_GROUPS)`.

### Tabelas (`/tables`)

Create/show/update/delete, trash/restore (individual e em lote), export CSV,
import de schema e de arquivo (`imports/file`, `.../analyze`, `.../commit`).
Guardas por `TA(ação)` (acesso à tabela por ação configurável).

### Campos e grupos de campo (`/tables/{slug}/fields`, `.../groups/{slug}/fields`)

Create/update/delete/trash/restore de campo, categorias de dropdown, campos
dentro de grupo (`GroupField*UseCase`).

### Relacionamentos (`/tables/{slug}/relationships`)

Create/update/delete da definição + links (criar, remover, reordenar).

### Linhas (`/tables/{slug}/rows`)

CRUD completo, bulk-update, auto-save (rascunho), trash/restore/reorder/
bulk-move-list, reação e avaliação, export CSV, import CSV, fórum
(mensagens + menção), import de arquivos em lote (`imports/files/archive`).
Linhas de grupo de campo têm sua própria fatia
(`/rows/{rowId}/groups/{groupSlug}`).

### Menu e páginas (`/menu`, `/pages`)

CRUD + reorder + trash/restore em lote, atrás de `Perm(MANAGE_MENU)`.

### Configuração e setup (`/setting`, `/setup`)

`GET /setting/public` (anônimo, allowlist), `GET/PUT /setting`
(`Perm(MANAGE_SETTINGS)`), `GET /setup/status` e
`PUT /setup/step/{admin,name,database,storage,logos,upload,paging,email}`
(atrás do segredo de bootstrap antes de concluído).

### Storage (`/storage`, `/files`)

Upload em partes (`POST /files`, `GET .../parts`, `POST .../complete`),
download, delete, migração de driver (`/storage/migration/{status,start,
cleanup}`, restrito a MASTER).

### Auditoria (`/logs`, `/error-logs`)

Restrito a `Role[MASTER, ADMINISTRATOR]`.

### Notificações (`/notifications`)

Paginação, contagem de não lidas, marcar lida, deletar — filtradas por
usuário.

### Ambientes (`/workspaces`)

Create/import/list/update/export/delete/restore/default, membros
(`GET/POST/PUT/DELETE`) — a maioria atrás de `Perm(MANAGE_WORKSPACE)` ou
`Role[MASTER]`.

### Extensões, chat, IA (`/extensions`, `/chat`, `/tables/{slug}/ai-access`, `/mcp`)

Toggle de extensão, configuração de escopo por tabela, upload de chat,
credenciais de acesso IA por tabela e o endpoint MCP (`POST /mcp`, atrás de
`AiAccess`).

## Extensões (pacote `core`, `apps`, `forms`)

| Extensão | Rotas | Toca rows |
| --- | --- | --- |
| `core/tools/clone-table` | `POST /tools/clone-table` | sim |
| `core/tools/tables-import-export` | export/import de tabela | sim |
| `core/tools/generate-test-data` | geração de dados fake | sim |
| `core/tools/installation-import` | conexões/planos/jobs de importação legada | sim |
| `core/tools/doc-transcription` | transcrição de documento (IA) | não |
| `core/tools/bpm-modeler` | modelador de processos BPM | cria tabelas |
| `core/plugins/row-access` | guard de linha por ACL | fragmentos de query |
| `core/modules/repository` | catálogo de apps remoto | sim |
| `apps/modules/dashboard` | estatísticas do dashboard | contagem |
| `apps/modules/senhas` | cofre de senhas | não |
| `apps/plugins/register-ip` | grava IP de quem agiu numa linha | sim |
| `forms/plugins/cascade-dropdown` | dropdown em cascata | sim |
| `forms/plugins/conditional-fields` | campos condicionais no formulário | não |

Veja [Extensões](../features/extensions.md) para o mecanismo completo.

## Services, hooks e middlewares (visão geral)

- **Fachadas por entidade** (`services/<x>/`): Table, Field, Row,
  Relationship, User, UserGroup, Storage, Setting, Email, Workspace,
  Notification, Menu, Extension, Logger, AiAccess.
- **Infra sem entidade**: `slug`, `identifier`, `date`, `search`,
  `type-guard`, `http-response`, `error-handler`, `redis`, `queue-worker`,
  `scheduler`, `socket-auth`, `script-execution`, `trash`, `password`,
  `csv-export`, `database`, `secret-crypto`, `rate-limit`, `http-outbound`
  (guarda de SSRF), `ai-access`, `domain-events`, `table-package`,
  `job-result`.
- **Hooks globais**: `onRequest` (workspace, content-disposition),
  `preSerialization` (presigned-url, secret-leak), `onResponse` (logger),
  `onSend` (error-log).

## Documento-fonte

Detalhe completo, com commits de cada mudança de comportamento e a lista de
regressões encontradas e corrigidas na auditoria de paridade, está em
`_doc-mapa-funcionalidades.md`, na raiz do repositório principal — mantido
sempre que uma rota, service ou extensão entra ou sai.
