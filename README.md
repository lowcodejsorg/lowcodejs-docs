# LowCodeJS

LowCodeJS é uma plataforma low-code para criação de aplicações com tabelas
dinâmicas, formulários, dashboards e automações. O usuário monta a estrutura de
dados pela interface — tabelas, campos, relacionamentos, permissões — e o
backend materializa tudo em runtime, sem gerar código.

## Stack

- **Backend**: Fastify 5 + TypeScript + MongoDB (driver nativo, sem ODM), com
  Postgres opcional para as tabelas de dados.
- **Frontend**: React 19 + TanStack Start (SSR) + TanStack Router/Query/Table +
  React Hook Form + Base UI + Tailwind CSS 4.
- **Autenticação**: JWT RS256 em cookies httpOnly.
- **Autorização**: RBAC com grupos customizáveis, hierarquia e capacidades por
  área.

## Principais capacidades

- **Tabelas dinâmicas**: criação de tabelas com campos configuráveis (texto,
  data, relacionamento, arquivo, grupo de campos, etc.), schema montado em
  runtime pelo `RowStore`.
- **Formulários e visualizações**: 9 estilos de visualização de tabela (lista,
  galeria, kanban, calendário, fórum, entre outros).
- **Dashboards**: gráficos e cartões de métrica sobre os dados das tabelas.
- **Automações**: scripts de usuário (`beforeSave`/`afterSave`/`onLoad`)
  executados em sandbox isolado, e um modelador BPM (extensão) para desenhar
  processos.
- **RBAC avançado**: grupos customizados com hierarquia (`encompasses`), 14
  capacidades de área e bindings de permissão por ação em tabela, campo e menu.
- **Multi-ambiente**: múltiplos *workspaces* isolados dentro da mesma
  instalação.
- **Extensões**: sistema de plugins/módulos/ferramentas para estender a
  plataforma sem alterar o core.
- **Self-host**: Docker Compose com serviços opcionais (Assistente IA,
  Postgres para as tabelas de dados).

## Por onde começar

- [Visão Geral](getting-started/overview.md) — o que a plataforma resolve e
  para quem.
- [Instalação](getting-started/installation.md) — como subir a plataforma via
  Docker Compose.
- [Primeiros Passos](getting-started/quickstart.md) — comandos essenciais de
  desenvolvimento.
