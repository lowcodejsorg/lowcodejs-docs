# Comandos

Todos os comandos essenciais do monorepo. Gerenciador de pacotes: **pnpm 12**
(`packageManager` no `package.json`; `npm i -g pnpm@12.4.1`). Cada pacote tem
seu próprio `pnpm-lock.yaml` — não há `package-lock.json`. O backend usa
`nodeLinker: hoisted` porque a imagem de produção copia `node_modules/` de um
artefato do CI.

## Setup

```bash
./setup.sh
```

## Desenvolvimento (Docker)

```bash
docker compose up -d
docker exec -it low-code-js-api pnpm seed
```

## Backend local

```bash
cd backend
pnpm install
pnpm dev              # dev mode (watch + SWC)
pnpm build            # tsc + tsup -> /build
pnpm start            # produção (build/bin/server.js)
pnpm seed             # seeders (permissions, grupos, usuários)
pnpm lint             # ESLint --fix
```

### Testes

```bash
pnpm test             # e2e (alias de test:e2e)
pnpm test:unit        # *.unit.spec.ts, não sobe Mongo nem kernel
pnpm test:e2e         # e2e (MongoDB real)
pnpm test:e2e:postgres # a mesma suíte com as tabelas de dados no Postgres
pnpm test:import      # suíte do import de instalação legada
pnpm test:coverage    # coverage (V8)
```

### Checagens estáticas

```bash
pnpm di:check         # resolve o grafo de DI; falha se alguma dep vier undefined
pnpm scope:check      # documento escopado sem ambiente
pnpm boot:check       # sobe o kernel E emite uma requisição
pnpm authz:check      # guardas de cada rota contra o baseline de autorização
pnpm check:guards     # di:check + scope:check + authz:check + boot:check
pnpm check            # tsc --noEmit + openapi:check + openapi:verify + sync:check
pnpm openapi:generate # regenera core/openapi/schema.generated.ts a partir dos controllers
pnpm postgres:sync    # garante as tabelas/pivôs de cada ambiente numa instalação Postgres
```

## Frontend local

```bash
cd frontend
pnpm install
pnpm dev              # Vite dev server, porta 5173
pnpm build            # build de produção (SSR + cliente)
pnpm start            # serve .output/server/index.mjs (Nitro)
pnpm test             # Vitest
pnpm lint             # Prettier + ESLint (auto-fix)
```

## Geração de boilerplate de extensão

```bash
pnpm make:extension                                   # interativo
pnpm make:extension --type plugin --id hello-table
pnpm make:extension --type tool --pkg core --id merge-tables --with-backend
```

Veja [Extensões](../features/extensions.md) para as flags completas.

## Publicação do catálogo de apps

Só na instalação que hospeda o repositório de apps (exige a tabela `Apps`
criada):

```bash
node scripts/seed-repository-apps.mjs --url http://localhost:3000 \
  --email <master> --password <senha>
```

## Credenciais

```bash
./credential-generator.sh
```

Gera JWT keys (RSA 2048, base64), `COOKIE_SECRET`,
`STORAGE_URL_SIGNING_KEY` e `PASSWORDS_ENCRYPTION_KEY` (preservada se já
existir).

## Scripts auxiliares (`scripts/`)

| Script | Uso |
| --- | --- |
| `make-extension.mjs` | gerador de boilerplate de extensão (plugin/módulo/tool) |
| `seed-repository-apps.mjs` | publica apps de exemplo no catálogo remoto |
| `validate-repository-apps.mjs` | valida os pacotes de app antes de publicar |
| `repository-apps.shared.mjs` | utilitários compartilhados dos dois scripts acima |
| `check-validator-parity.mjs` | confere paridade entre validators do backend e do frontend |
| `backup.sh` | rotina de backup |

## Commits

Conventional Commits atômicos em pt-BR (`refactor(escopo): ...`,
`fix(escopo): ...`), cobrados por `commitlint`.
