# Arquitetura — Visão Geral

## Estrutura do monorepo

```
lowcodejs/
├── backend/              # API Fastify + TypeScript + MongoDB
│   └── CLAUDE.md         # Arquitetura detalhada do backend
├── frontend/             # React + TanStack Start (SSR)
│   └── CLAUDE.md         # Arquitetura detalhada do frontend
├── scripts/              # make-extension + seed do repositório de apps
├── .github/workflows/    # CI/CD (build, test, push :latest no Docker Hub)
├── docker-compose.yml              # Desenvolvimento (core + profiles opcionais)
├── docker-compose.oficial.yml      # Self-host mínimo via imagens :latest
├── install.md             # Guia de instalação
├── setup.sh                # Script de configuração inicial
└── credential-generator.sh # Gera JWT keys e cookie secret
```

## Serviços

### Core (sempre sobem)

| Serviço | Tecnologia      | Porta | Descrição                |
| ------- | ---------------- | ----- | -------------------------- |
| mongo   | MongoDB           | 27017 | Banco de dados               |
| redis   | Redis 7 Alpine    | 6379  | Cache                         |
| api     | Fastify / Node 24 | 3000  | Backend REST + WebSocket     |
| app     | React / Nitro     | 5173  | Frontend SSR                  |

### Opcionais (via Docker Compose profiles)

| Serviço  | Profile         | Tecnologia  | Porta | Descrição                                      |
| -------- | --------------- | ------------ | ----- | ------------------------------------------------ |
| mcp      | `ai`            | MCP Server   | 3001  | Assistente IA (Chat + OpenAI)                     |
| postgres | `data-postgres` | Postgres 16  | 5432  | Banco das tabelas de dados (escolha do wizard)     |

```bash
# Core apenas
docker compose up -d

# Com Assistente IA
docker compose --profile ai up -d

# Com Postgres para as tabelas de dados
docker compose --profile data-postgres up -d
```

## Diagrama textual da arquitetura

```
Cliente HTTP / Browser
        │
        ▼
  Frontend (React + TanStack Start, SSR) ── porta 5173
        │  fetch/REST + Socket.IO
        ▼
  Backend (Fastify 5 + TypeScript) ── porta 3000
        │
        ├── Middleware (auth, permission, table-access, rate-limit, ...)
        ├── Controller → Validator (VineJS) → Use Case → Either<Erro, Sucesso>
        ├── Service (contrato + impl, DI por fastify-decorators)
        │
        ├── Store da entidade (ScopedCollection) ──► MongoDB (DB_DATABASE)
        │     users, tables, fields, settings, relacionamentos...
        │
        └── RowStore(table) — montado em runtime a partir do _schema
              ├──► MongoDB (DB_DATA_DATABASE, um database por ambiente)
              └──► PostgreSQL (DATA_POSTGRES_URL, schema por ambiente) — opcional

  Redis ── cache, rate limit, filas (BullMQ: email, import CSV/planilha,
           envio em lote, migração/otimização de storage, import de instalação)

  MCP Server (opcional, profile `ai`) ── Assistente IA via Chat + OpenAI
```

Um único processo Node para a API, um `MongoClient` com **dois databases**
(sistema e dados), um Postgres opcional só para as tabelas de dados, e um
Redis compartilhado pela instalação inteira.

## Banco das tabelas de dados: Mongo ou Postgres

O **sistema** (users, tables, fields, settings, links...) vive sempre no
MongoDB. As **tabelas de dados** (tables × fields × rows do low-code) vivem no
Mongo **ou** no Postgres — escolha da etapa `database` do Setup Wizard, gravada
uma vez no Setting de instância (`DATA_DRIVER`) e **imutável**. Instalação
existente é Mongo (migration 60). Detalhe completo em
[Camada de Dados](data-layer.md).

## Documentação relacionada

- [Backend](backend.md) — Fastify, DI, Either pattern, use-cases, extensões.
- [Frontend](frontend.md) — React 19, TanStack, formulários, design system.
- [Camada de Dados](data-layer.md) — Mongo vs Postgres para tabelas de dados.
- [Mapa de Funcionalidades](feature-map.md) — rotas, use-cases, services,
  extensões.
- [Decisões de Arquitetura](decisions.md) — ADRs do repositório.
