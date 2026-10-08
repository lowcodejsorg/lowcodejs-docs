# Instalação

Guia de self-host via Docker Compose (recomendado) ou desenvolvimento local.

## Pré-requisitos

- **Docker** e **Docker Compose**
- **Node.js 24** e **pnpm 12** (`npm i -g pnpm@12.4.1`), só para desenvolvimento
  local sem container
- **Git Bash** (obrigatório no Windows, para rodar os scripts `.sh`)

## Configuração inicial

Na raiz do projeto:

```bash
chmod +x ./setup.sh
./setup.sh
```

O script `setup.sh`:

- Cria `.env` a partir de `.env.example`
- Cria `.env.test` a partir de `.env.test.example`
- Gera as credenciais JWT automaticamente (via `credential-generator.sh`)
- Separa as variáveis em `backend/.env` (sem `VITE_*`) e `frontend/.env` (só
  `VITE_*`)

Rodar `setup.sh` de novo **sobrescreveria** o `.env` já configurado — por isso
ele se recusa a recriar um `.env` existente, salvo com `./setup.sh --force`.

> Depois de a aplicação subir pela primeira vez, o **Setup Wizard** na UI guia
> a configuração de branding, logos, locale, storage (local/S3), upload,
> paginação, SMTP e Assistente IA — tudo persistido no documento `Setting` do
> MongoDB. Não há variáveis de ambiente para esses itens.

### Gerar credenciais manualmente

```bash
./credential-generator.sh
```

Gera `JWT_PRIVATE_KEY`/`JWT_PUBLIC_KEY` (RSA 2048, base64), `COOKIE_SECRET`,
`STORAGE_URL_SIGNING_KEY` e `PASSWORDS_ENCRYPTION_KEY`. A
`PASSWORDS_ENCRYPTION_KEY` é **preservada** se já existir — regerá-la tornaria
ilegível todo segredo já cifrado no Setting e no cofre de senhas.

**Nunca** use as chaves padrão/de exemplo em produção.

## Opção 1: Docker (recomendado)

### Subir os serviços essenciais

```bash
docker compose up -d
```

Sobe MongoDB, Redis, Backend (API) e Frontend (App) — os serviços **core**.

### Seeds do banco

```bash
docker exec -it low-code-js-api pnpm seed
```

### Acessos

| Serviço  | URL                                  |
| -------- | ------------------------------------- |
| Frontend | http://localhost:5173                 |
| Backend  | http://localhost:3000                 |
| Docs API | http://localhost:3000/documentation   |

## Serviços opcionais

### Assistente IA (Chat)

```bash
docker compose --profile ai up -d
```

Depois de subir o container, configure a **chave OpenAI** e ative o toggle em
**Configurações** na UI (usuário MASTER).

Para sempre subir com IA, defina no `.env`:

```env
COMPOSE_PROFILES=ai
```

### PostgreSQL para as tabelas de dados

O **sistema** (usuários, tabelas, campos, configurações...) vive sempre no
MongoDB. As **tabelas de dados** criadas no low-code podem viver no PostgreSQL
em vez do Mongo — uma tabela SQL por tabela, coluna tipada por campo, FK nos
relacionamentos, tabela pivô nos N:N.

```bash
docker compose --profile data-postgres up -d
```

Com o serviço de pé e `DATA_POSTGRES_URL` preenchida no `.env`, a etapa "Banco
de dados" do Setup Wizard passa a oferecer a escolha entre MongoDB e
PostgreSQL. **A escolha é feita uma vez**, em instalação nova, e não muda
depois — não há migração de um banco para o outro. Veja
[Camada de Dados](../architecture/data-layer.md).

### Armazenamento S3

Não exige container adicional — configurado no Setup Wizard ou em
**Configurações** na UI (MASTER). Sem S3 ativado, o storage usa filesystem
local (`_storage/`).

## Opção 2: Desenvolvimento local

Subir só Mongo e Redis:

```bash
docker compose up -d mongo redis --build
```

Backend:

```bash
cd backend
pnpm install
pnpm seed
pnpm dev
```

Frontend (em outro terminal):

```bash
cd frontend
pnpm install
pnpm dev
```

## Variáveis de ambiente

O `setup.sh` já copia `.env.example` → `.env`. Manualmente:

```bash
cp .env.example .env
cp .env.test.example .env.test
```

Os defaults em `.env.example` apontam para hosts nativos (`127.0.0.1`,
`localhost`) — cenário padrão de dev local com mongo/redis em Docker e
backend/frontend rodando direto na máquina. Quando o stack inteiro sobe via
`docker compose up -d`, o próprio compose sobrescreve `DATABASE_URL`,
`REDIS_URL` e `MCP_SERVER_URL` para os hosts internos da rede Docker (`mongo`,
`redis`, `mcp`). Não é preciso editar nada ao alternar entre os dois modos.

### Principais variáveis

| Variável            | Descrição                              | Padrão                    |
| ------------------- | --------------------------------------- | -------------------------- |
| `NODE_ENV`          | Ambiente de execução                    | `development`               |
| `PORT`              | Porta do backend                        | `3000`                      |
| `DB_USERNAME`       | Usuário do MongoDB                      | `lowcodejs`                 |
| `DB_PASSWORD`       | Senha do MongoDB                        | `lowcodejs`                 |
| `DB_DATABASE`       | Nome do banco system                    | `lowcodejs`                 |
| `DB_DATA_DATABASE`  | Nome do banco data (dinâmicas)          | `lowcodejs_data`            |
| `DATABASE_URL`      | URL de conexão MongoDB                  | `mongodb://...`             |
| `DATA_POSTGRES_URL` | URL do PostgreSQL das tabelas de dados (opcional; habilita a escolha no wizard) | vazio |
| `APP_SERVER_URL`    | URL pública do backend                  | `http://localhost:3000`     |
| `APP_CLIENT_URL`    | URL pública do frontend                 | `http://localhost:5173`     |

> **Configurações de domínio** (branding, locale, logos, upload, paginação,
> storage S3, SMTP, IA) **não** vivem em `.env` — são editadas via UI
> `/settings` (usuário MASTER) ou pelo Setup Wizard no primeiro acesso.

### Assistente IA (opcional)

| Variável          | Descrição            | Padrão                  |
| ----------------- | --------------------- | ------------------------ |
| `MCP_SERVER_URL`  | URL do servidor MCP    | `http://mcp:3000/mcp`    |

A chave da API OpenAI e o toggle "Habilitar Assistente IA" são configurados
pelo usuário MASTER em Configurações (`/settings`), não via variável de
ambiente.

## Limpeza completa (reset)

```bash
# Parar os containers
docker compose --profile s3 --profile ai down

# Remover containers, imagens e cache
docker system prune -f -a --volumes

# Remover o volume do MongoDB
docker volume rm low-code-js_mongo-volume
```

**Atenção**: isso apaga todos os dados do banco.

## Troubleshooting

**Erro de permissão no `setup.sh`**:

```bash
chmod +x ./setup.sh ./credential-generator.sh
```

**Container não encontrado**: confira `docker ps`.

**Erro de conexão com MongoDB**: o `.env` vem com `DATABASE_URL` apontando
para `127.0.0.1:27017` (host nativo); dentro de `docker compose up -d` o
compose sobrescreve para o host interno `mongo`. Em testes (`pnpm test:e2e`),
`127.0.0.1` é obrigatório — testes nunca rodam em container.
