# Primeiros Passos

Comandos essenciais para o dia a dia de desenvolvimento. Gerenciador de
pacotes: **pnpm 12** (`npm i -g pnpm@12.4.1`). Cada pacote (`backend/`,
`frontend/`) tem seu próprio `pnpm-lock.yaml`.

## Setup inicial

```bash
./setup.sh
```

## Subir o ambiente completo (Docker)

```bash
docker compose up -d
docker exec -it low-code-js-api pnpm seed
```

Acessos: Frontend em `http://localhost:5173`, Backend em
`http://localhost:3000`, Docs da API em `http://localhost:3000/documentation`.

## Backend local (sem Docker para a API)

```bash
cd backend
pnpm install
pnpm dev          # dev mode (watch + SWC)
```

Outros comandos úteis do backend:

```bash
pnpm seed         # seeders (permissions, grupos, usuários)
pnpm build        # tsc + tsup -> /build
pnpm lint         # ESLint --fix
pnpm start        # produção (build/bin/server.js)
```

## Frontend local

Em outro terminal:

```bash
cd frontend
pnpm install
pnpm dev          # Vite dev server, porta 5173
```

```bash
pnpm build        # build de produção (SSR + cliente)
pnpm lint         # Prettier + ESLint --fix
```

## Testes

```bash
# Backend
cd backend
pnpm test             # e2e (alias de test:e2e)
pnpm test:unit        # *.unit.spec.ts — não sobe Mongo nem kernel
pnpm test:e2e         # e2e (MongoDB real, 1 worker)
pnpm test:e2e:postgres # a mesma suíte e2e com as tabelas de dados no Postgres

# Frontend
cd frontend
pnpm test             # Vitest
```

Veja [Testes](../contributing/testing.md) para o roteiro completo (containers
necessários, variáveis de `.env.test`, interpretação dos resultados).

## Gerar boilerplate de extensão

```bash
pnpm make:extension                                    # interativo
pnpm make:extension --type plugin --id hello-table     # via flags
```

Gera a estrutura de uma extensão (`PLUGIN`, `MODULE` ou `TOOL`) nos dois lados
do monorepo. Veja [Extensões](../features/extensions.md).

## Publicar os apps de exemplo no catálogo

Só na instalação que hospeda o repositório de apps (exige a tabela `Apps`
criada):

```bash
node scripts/seed-repository-apps.mjs --url http://localhost:3000 \
  --email <master> --password <senha>
```

## Checks estáticos do backend

```bash
cd backend
pnpm di:check       # resolve o grafo de DI
pnpm scope:check    # documento escopado sem ambiente
pnpm boot:check     # sobe o kernel e emite uma requisição
pnpm authz:check    # guardas de cada rota contra o baseline de autorização
pnpm check:guards   # di:check + scope:check + authz:check + boot:check
pnpm check          # tsc --noEmit + openapi:check + openapi:verify + sync:check
pnpm openapi:generate # regenera o schema OpenAPI a partir dos controllers
```

## Próximos passos

- [Arquitetura — Visão Geral](../architecture/overview.md)
- [Arquitetura — Backend](../architecture/backend.md)
- [Arquitetura — Frontend](../architecture/frontend.md)
