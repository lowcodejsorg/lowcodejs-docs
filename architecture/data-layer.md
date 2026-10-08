# Arquitetura — Camada de Dados

## Duas categorias de dado

- **Sistema** (users, tables, fields, settings, definições de relacionamento,
  menus...) — vive **sempre** no MongoDB, no database `DB_DATABASE`.
- **Tabelas de dados** (tables × fields × rows criadas no low-code) — vivem no
  MongoDB **ou** no PostgreSQL, conforme a escolha feita na etapa `database`
  do Setup Wizard.

A escolha é gravada no Setting de instância (`DATA_DRIVER`) e é **imutável**:
não há migração de um banco para o outro depois. Instalação que já existia
antes dessa funcionalidade é Mongo para sempre (migration 60 carimba
`'mongo'`). A tela **Configurações** mostra qual banco está em uso, só
leitura. Veja a decisão completa na [ADR 0007](decisions.md).

## MongoDB (padrão)

Um `MongoClient`, dois databases:

- **System** (`DB_DATABASE`): as collections dos 23 models de
  `application/model/` (User, UserGroup, Permission, Table, Field, Storage,
  Setting, Workspace, WorkspaceMember, Extension, Notification, etc). É o
  database default do client.
- **Data** (`DB_DATA_DATABASE`, e `${DB_DATA_DATABASE}_<slug>` por ambiente):
  uma collection por `table.slug`. `dataDb()` escolhe o database pelo escopo
  corrente (`AsyncLocalStorage`).

O backend não usa ODM (Mongoose foi removido — ver ADR 0006). Cada entidade
tem uma **definição** (`defineModel`) e um **store** (`ModelStore<T>`) sobre
`ScopedCollection`. As rows das tabelas do low-code passam por `RowStore`,
montado em runtime a partir do `_schema` da tabela.

## PostgreSQL (opcional, para as tabelas de dados)

Quando a instalação escolhe Postgres, a forma é **relacional de verdade**:

- Schema por ambiente (`workspace.dataDatabase`), tabela por slug
- Coluna real por campo — não JSONB por row
- `_id text` (PK), hex de 24 gerado pelo backend
- `FIELD_GROUP` vira **tabela filha** com FK `ON DELETE CASCADE`
- Relacionamento 1:1/1:N vira **FK real** com o `onDelete` da definição
  (CASCADE/SET NULL/RESTRICT); 1:1 ganha UNIQUE parcial
- Relacionamento N:N vira **tabela pivô** por definição
- `INTEGER`/`DECIMAL` viram `bigint`/`numeric` (a API segue devolvendo string)
- `showInFilter` → índice btree/GIN; `IS_UNIQUE` → UNIQUE parcial

### Mapa de tipos

| Tipo de campo | Coluna Postgres | Índice/constraint |
| --- | --- | --- |
| TEXT_SHORT (padrão) | `text` | btree se `showInFilter`; UNIQUE parcial se `IS_UNIQUE` |
| TEXT_SHORT `INTEGER` | `bigint` | btree |
| TEXT_SHORT `DECIMAL` | `numeric` | btree |
| TEXT_LONG | `text` | — |
| DATE | `timestamptz` | btree se `showInFilter` |
| DROPDOWN / CATEGORY | `text[]` | GIN |
| FILE / USER / USER_GROUP / REACTION / EVALUATION | `text[]` | GIN, sem FK |
| RELATIONSHIP OWNS_FK | `text REFERENCES <alvo>` | btree; UNIQUE parcial em 1:1 |
| RELATIONSHIP REVERSE | `text[]` | — |
| RELATIONSHIP PIVOT (N:N) | tabela própria com `sourceId`/`targetId` | btree em cada lado |
| FIELD_GROUP | tabela filha com FK `ON DELETE CASCADE` | btree em `_parent` |

### Arquitetura do driver

```
services/table/model-builder/
  model-builder-contract.service.ts   # fachada: build/driver/schemaEpoch/invalidate
  row-driver.contract.ts              # 1 contrato de driver
  internal/mongo-row-driver/          # impl Mongo
  internal/postgres-row-driver/       # impl Postgres (DDL, tradutor, store)
    column-map.ts        # IField/_schema → ColumnSpec
    sql-translator.ts    # QueryFilter/UpdateOperators → SQL + params
    ddl.ts               # CREATE SCHEMA/TABLE, ADD/ALTER COLUMN, FK, índices

services/relationship/store/
  internal/mongo-link-store/
  internal/postgres-link-store/       # tabela pivô no schema do workspace

services/database/
  data-driver-config/   # lê DATA_DRIVER do Setting de instância (imutável)
  data-transaction/     # PG: BEGIN/COMMIT via AsyncLocalStorage; Mongo: compensação
```

O contrato corta no nível da **collection**, com a gramática `QueryFilter`/
`UpdateOperators` que o domínio já falava — os ~20 consumidores de
`model.collection()` não mudam. Mongo vira wrapper do que já existia; Postgres
vira tradutor `QueryFilter → SQL` sobre colunas tipadas.

### O que o Postgres não traduz

Operador fora da gramática suportada **lança** (nunca degrada em silêncio):
`$elemMatch`, `$all`, `$size`, `$type`, `$mod`, `$expr`, `$where`, `$text` e
`$regex` com flags além de `i` respondem 501 `PG_UNSUPPORTED_OPERATOR`; sort/
distinct em path de grupo ou desconhecido respondem 501
`PG_UNSUPPORTED_PATH`.

Outras diferenças de comportamento (não-bugs, documentadas):

| Situação | Mongo | Postgres |
| --- | --- | --- |
| `bulkWrite`/`insertMany` ordered com falha no meio | mantém operações anteriores | ROLLBACK de tudo (transação) |
| `modifiedCount` | só o que mudou | igual a `matchedCount` |
| Ordem de texto em `sort` | binária (UTF-8) | collation do banco |
| `updatedAt` em update | relógio da aplicação | `now()` do banco |
| `ON DELETE` de relacionamento | decidido pela aplicação | FK real (RESTRICT/CASCADE de verdade) |

A suíte e2e roda nos dois bancos: `pnpm test:e2e` (Mongo) e
`pnpm test:e2e:postgres` (`TEST_DATA_DRIVER=postgres`). Isso é o que prova a
paridade — 137 arquivos / 486+ testes verdes nos dois bancos.

## Configuração e wizard

- `DATA_POSTGRES_URL` no `.env` habilita o pool (`@fastify/postgres`,
  registrado só se a variável existir) e faz a etapa `database` do Setup
  Wizard oferecer a escolha
- Sem a variável, o wizard só oferece Mongo
- `PUT /setting` descarta qualquer tentativa de mudar `DATA_DRIVER` — o
  prefixo `DATA_` é instance-only
- `docker-entry-point.sh` roda `sync-data-schema` no boot quando
  `DATA_DRIVER=postgres`, garantindo tabelas/pivôs de cada ambiente

## Documentos-fonte

Implementação detalhada e histórico de fases em
`_doc-postgres-data-driver.md` e `_doc-mongo-driver.md`, na raiz do
repositório principal.
