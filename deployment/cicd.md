# CI/CD

Dois caminhos de deploy, independentes entre si.

## GitHub Actions (branch `main`)

Workflows em `.github/workflows/`. Na branch `main`:

1. Roda os testes (backend e frontend)
2. Builda as imagens Docker
3. Publica `marcosjhollyfer/lowcodejs-api:latest` e
   `marcosjhollyfer/lowcodejs-app:latest` no Docker Hub

Essas imagens `:latest` são o que alimenta o
`docker-compose.oficial.yml` — quem faz self-host sem Coolify sobe
exatamente essa tag.

Não há backport de correção para uma tag anterior: toda correção de segurança
sai em `main` e vira `:latest` (ver
[Política de Segurança](../security/hardening.md)).

## Coolify (branches `main` e `develop`)

O Coolify observa as duas branches e faz build/deploy automático a partir de:

- `backend/Dockerfile-coolify`
- `frontend/Dockerfile-coolify`

Variáveis de ambiente são configuradas pelo painel do Coolify:

- **Frontend**: build arg `VITE_API_BASE_URL`; ENV de runtime `SERVER_URL` e
  `SERVER_API_URL`
- **Backend**: as variáveis de infraestrutura descritas em
  [Instalação](../getting-started/installation.md)

Logos, branding, locale, storage, SMTP e configuração de IA **não** são
variáveis de ambiente — vivem no documento `Setting` do MongoDB e são
editados via UI `/settings` depois do deploy (ver
[Configurações](../features/settings.md)).

## Implicação prática para quem desenvolve

`main` e `develop` **disparam deploy real** (Coolify observa as duas). Por
isso, trabalho experimental ou uma correção de segurança em andamento usa uma
branch própria e só sobe para `develop`/`main` com autorização explícita —
convenção seguida, por exemplo, durante o hardening de segurança
(`security/hardening-2026-08`, ver [Hardening](../security/hardening.md)).
