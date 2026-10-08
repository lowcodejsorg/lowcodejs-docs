# Configurações

O usuário **MASTER** configura a instância pela interface, no menu
Configurações (`/settings`). Tudo fica persistido no documento `Setting` do
MongoDB — nunca em variável de ambiente, salvo a connection string de
infraestrutura em si.

## O que é configurável pela UI

### Repositório de Apps

Toggle ativar/desativar + endereço da instalação que publica o catálogo de
apps de exemplo. O endereço é o da **API**, não do site (default
`https://api.net.lowcodejs.org`).

### Armazenamento S3

Toggle ativar/desativar + endpoint, região, bucket e credenciais (o usuário
configura seu próprio S3 externo). Quando desativado, o storage usa
filesystem local (`_storage/`).

Quando o MASTER troca o driver (local ↔ S3), os arquivos existentes ficam
órfãos no driver antigo. O recurso de **migração de storage** copia os
arquivos para o driver atual em background, com progresso em tempo real via
WebSocket (namespace `/storage-migration`):

| Endpoint | Método | Descrição |
| --- | --- | --- |
| `/storage/migration/status` | GET | Contagens por driver/status, job ativo |
| `/storage/migration/start` | POST | Enfileira job de migração |
| `/storage/migration/cleanup` | POST | Apaga arquivos do driver antigo |

Restrito a MASTER. A migração usa BullMQ sobre Redis; o worker retoma de
onde parou em caso de restart, pula arquivos já migrados (idempotente) e
tenta cada arquivo até 3 vezes com backoff.

### Assistente IA

Toggle ativar/desativar + chave OpenAI. Quando desativado, o chat fica
indisponível. O MCP server (serviço opcional, profile `ai` do Docker Compose)
precisa estar no ar para o chat funcionar.

### Banco das tabelas de dados

**Só leitura** — mostra se a instalação usa Mongo ou Postgres. Escolhido uma
única vez no Setup Wizard e **imutável** depois (ver
[Camada de Dados](../architecture/data-layer.md) e
[ADR 0007](../architecture/decisions.md)).

### Demais (via Setup Wizard, também editáveis depois)

Branding, logos, locale, upload (limites e extensões aceitas), paginação,
SMTP (host/porta/usuário/senha/remetente).

## Segredos no Setting

Seis campos de segredo (`STORAGE_ACCESS_KEY`, `STORAGE_SECRET_KEY`,
`EMAIL_PROVIDER_PASSWORD`, `OPENAI_API_KEY`, `MCP_SERVER_TOKEN`,
`LLM_API_KEY`) são **cifrados em repouso** desde a
[ADR 0009](../architecture/decisions.md) — AES com HKDF-SHA-256 e AAD por
campo, chave derivada de `PASSWORDS_ENCRYPTION_KEY` (com fallback para
`COOKIE_SECRET`). A API responde esses campos como indicador booleano de
"configurado", nunca com o valor em claro — inclusive para quem tem
`MANAGE_SETTINGS`.

## Email (SMTP)

Não é variável de ambiente. Campos do documento `Setting`:
`EMAIL_PROVIDER_HOST`, `EMAIL_PROVIDER_PORT`, `EMAIL_PROVIDER_USER`,
`EMAIL_PROVIDER_PASSWORD`, `EMAIL_PROVIDER_FROM` (todos nullable). Lido a
cada envio, sem cache. Se faltar credencial essencial, o envio retorna
`{ success: false, message: 'SMTP não configurado' }` sem lançar erro.
