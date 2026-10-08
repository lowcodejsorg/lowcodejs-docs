# Docker

LowCodeJS tem dois arquivos de Docker Compose, para dois cenários diferentes.

## `docker-compose.yml` — desenvolvimento

Builda as imagens localmente a partir dos `Dockerfile-local` de cada pacote,
monta o código como volume (hot reload) e expõe Mongo/Redis/Postgres também
no host (`127.0.0.1`, para a suíte e2e rodar fora do container).

### Serviços core (sempre sobem)

| Serviço | Imagem | Porta | Observação |
| --- | --- | --- | --- |
| `mongo` | `mongo:8.2` (fixada — tags móveis `8.x` têm bug de boot em kernel Linux recente) | 127.0.0.1:27017 | volume `mongo-volume` |
| `redis` | `redis:7-alpine` | 127.0.0.1:6379 | volume `redis-volume` |
| `api` | build local (`backend/Dockerfile-local`) | `${APP_SERVER_PORT}:3000` | `start_period: 180s` — migrations+seeders rodam antes do healthcheck responder |
| `app` | build local (`frontend/Dockerfile-local`) | `${APP_CLIENT_PORT}:5173` | não recebe `env_file: .env` — só as variáveis `VITE_*`/`SERVER_*` explícitas |

### Serviços opcionais (profiles)

| Serviço | Profile | Imagem | Porta |
| --- | --- | --- | --- |
| `postgres` | `data-postgres` | `postgres:16-alpine` | 127.0.0.1:5432 |
| `mcp` | `ai` | `marcosjhollyfer/lowcodejs-mcp:latest` | `${MCP_PORT:-3001}:3000` |

```bash
docker compose up -d                              # core
docker compose --profile ai up -d                 # + Assistente IA
docker compose --profile data-postgres up -d      # + Postgres para tabelas de dados
```

A `api` depende de `postgres` com `required: false`: sobe sem o serviço
quando o profile está desligado, e espera-o saudável quando está ligado — caso
contrário o `sync-data-schema` do entrypoint corria antes do banco aceitar
conexão.

## `docker-compose.oficial.yml` — self-host via imagens publicadas

Usa as imagens `:latest` publicadas no Docker Hub
(`marcosjhollyfer/lowcodejs-api`, `marcosjhollyfer/lowcodejs-app`) em vez de
buildar localmente. Pensado para quem faz self-host sem Coolify, só com
`docker compose`.

Diferenças em relação ao de desenvolvimento:

- `DB_PASSWORD` é **obrigatória** (`${DB_PASSWORD:?Defina DB_PASSWORD no .env}`)
  — o compose de dev tem senha default (`lowcodejs`), o oficial recusa subir
  sem uma senha explícita.
- Volumes nomeados para storage e para o `.output/public` do frontend
  (`lowcodejs-storage`, `lowcodejs-app-public`).
- Não expõe porta de Mongo/Redis/Postgres para o host — só a rede interna.
- `IMAGE_TAG` (default `latest`) controla a tag puxada do Docker Hub.

```bash
docker compose -f docker-compose.oficial.yml up -d
./setup.sh --frontend-url   # ajusta a URL da API embutida no bundle do frontend
```

O `setup.sh --frontend-url` existe porque o bundle do frontend é buildado com
`VITE_API_BASE_URL` fixo; ao usar a imagem publicada (que não conhece o
`APP_SERVER_URL` do seu host), o script substitui a URL dentro dos arquivos
JS já gerados, direto no volume.

## Healthchecks e tempo de boot

O `api` tem `start_period: 180s` nos dois composes: o
`docker-entry-point.sh` roda todas as migrations e os seeders **antes** de o
servidor escutar. Numa base com dados isso passa de 30 segundos com folga — e
é por isso que o período de graça é alto: sem ele, o healthcheck marcava
`unhealthy` no meio do trabalho e o `depends_on: service_healthy` do frontend
abortava, dando a impressão de que o deploy tinha falhado.

## Volumes

| Volume | Conteúdo |
| --- | --- |
| `mongo-volume` / `lowcodejs-mongo-data` | Dados do MongoDB |
| `redis-volume` / `lowcodejs-redis-data` | Dados do Redis (AOF no oficial) |
| `postgres-volume` / `lowcodejs-postgres-data` | Dados do Postgres (só com o profile) |
| `lowcodejs-storage` | Arquivos enviados (storage local, só no compose oficial) |
| `lowcodejs-app-public` | Build estático do frontend (`.output/public`) |

## Rede

Uma única rede bridge (`low-code-js-network` / `lowcodejs-network`) para todos
os serviços — a comunicação interna usa o nome do serviço como host
(`mongo`, `redis`, `api`, `postgres`, `mcp`).
