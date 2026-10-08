# Tabelas Dinâmicas

O núcleo do low-code: o usuário desenha a tabela pela UI e o backend monta o
schema em runtime, sem geração de código nem deploy.

## Como funciona

Cada tabela guarda um `_schema` (campo `Mixed` no documento `Table`) que é
convertido em runtime pelo `ModelBuilderContractService.build(table)` num
`RowStore` — a collection da tabela no database de dados (Mongo) ou a tabela
relacional equivalente (Postgres), com cast de tipo, defaults e os scripts
`beforeSave`/`afterSave`. É esse mecanismo que permite criar tabelas novas na
UI sem reiniciar o servidor.

Implementação em `backend/application/services/table/model-builder/`
(contrato de driver + `internal/mongo-row-driver/` +
`internal/postgres-row-driver/`). Veja
[Camada de Dados](../architecture/data-layer.md) para o detalhe de como cada
tipo de campo vira coluna no Postgres.

## Tipos de campo

`E_FIELD_TYPE` — configuráveis: TEXT_SHORT, TEXT_LONG, DROPDOWN, DATE,
RELATIONSHIP, FILE, FIELD_GROUP, REACTION, EVALUATION, CATEGORY, USER,
USER_GROUP, HTML_CONTENT. Nativos (sempre presentes): CREATOR, IDENTIFIER,
CREATED_AT, UPDATED_AT, UPDATER, TRASHED_AT, STATUS.

`TEXT_SHORT` aceita formatos adicionais (`E_FIELD_FORMAT`): ALPHA_NUMERIC,
INTEGER, DECIMAL, URL, EMAIL, PASSWORD, PHONE, CNPJ, CPF, além de formatos de
texto longo (RICH_TEXT, PLAIN_TEXT) e formatos de data.

Validações de campo (`E_FIELD_VALIDATION`, 16 regras): NOT_EMPTY, IS_EMAIL,
IS_NUMERIC, IS_ALPHA_NUMERIC, IS_IN_RANGE, IS_IBAN, IS_NOT, IS_URL, IS_PHONE,
IS_CPF, IS_CNPJ, IS_UNIQUE, ARE_UNIQUE_VALUES, EMAIL_EXISTS, USER_EXISTS,
GROUP_EXISTS — camada única em `services/field/field-validation/rules/`.

## Relacionamentos

Modelados pela UI entre duas tabelas, com cardinalidade 1:1, 1:N ou N:N.
No Mongo, materializados como link top-level (`relationshipId` + campo
espelho). No Postgres, viram FK real (1:1/1:N, com o `onDelete` da definição:
CASCADE, SET NULL ou RESTRICT) ou tabela pivô (N:N).

## Grupos de campo (`FIELD_GROUP`)

Subdocumento repetível dentro de uma row (ex.: itens de um pedido). No Mongo é
um array embutido; no Postgres vira **tabela filha** com FK
`ON DELETE CASCADE` para a tabela pai.

## Estilos de visualização

`E_TABLE_STYLE` — 9 estilos: LIST, GALLERY, DOCUMENT, CARD, MOSAIC, KANBAN,
FORUM, CALENDAR, GANTT. Cada um tem seu conjunto de componentes no frontend
(`frontend/src/routes/_private/tables/$slug/`).

## Scripts de usuário (automação por tabela)

Cada tabela pode ter código JavaScript nos momentos `beforeSave`, `afterSave`
e `onLoad`, executado no servidor dentro de um sandbox isolado
(`isolated-vm`, heap de 128 MB, timeout de 5s). APIs disponíveis dentro do
script:

| API | Métodos | Descrição |
| --- | --- | --- |
| `field` | `get(slug)`, `set(slug, value)`, `getAll()`, `getLabel(slug, value?)` | Leitura/escrita de campos do registro |
| `context` | `action`, `moment`, `userId`, `isNew`, `appUrl`, `table`, `reentrant`, `previous` | Contexto de execução (read-only) |
| `email` | `send(to[], subject, body)`, `sendTemplate(to[], subject, message, data?)` | Envio de email |
| `users` | `resolve(ids)`, `emails(ids)` | Resolve ids de campos USER/CREATOR |
| `notify` | `send({ userIds, title, body?, action?, source? })` | Notificações in-app + socket |
| `utils` | `today()`, `now()`, `formatDate(date, format?)`, `sha256(text)`, `uuid()` | Utilitários |
| `console` | `log()`, `warn()`, `error()` | Logs capturados e retornados |

Retorno: `ExecutionResult { success, error?, logs[] }`, com tipos de erro
`syntax`, `runtime`, `timeout`, `unknown`.

Veja [Modelo de Ameaças](../security/threat-model.md) para o desenho de
segurança do sandbox e por que nenhuma referência do host pode atravessar a
fronteira.

## Permissões de tabela, campo e linha

Controle granular descrito em [RBAC](rbac.md): bindings por ação
(`table.permissions`), por contexto de campo (`field.permissions`), membros
da tabela (`table.members`) com perfis fixos, e um guard de linha opcional
via extensão (`core/plugins/row-access`).

## Importação e exportação

- Export/import de tabela inteira (estrutura + dados), extensão
  `core/tools/tables-import-export`
- Import de arquivo/planilha para criar tabela (`POST /tables/imports/file`)
- Export CSV de linhas (`GET /rows/exports/csv`)
- Import CSV de linhas (`POST /rows/imports/csv`)
- Clonagem de tabela (extensão `core/tools/clone-table`)
- Geração de dados de teste (extensão `core/tools/generate-test-data`)
