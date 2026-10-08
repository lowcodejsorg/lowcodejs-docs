# Visão Geral

LowCodeJS é uma plataforma low-code self-hostable para criação de aplicações
com tabelas dinâmicas, formulários, dashboards e automações. O usuário
configura a estrutura de dados pela UI e o backend monta tudo em runtime, sem
etapa de geração/compilação de código específico da aplicação.

## Modelo de uso

- O usuário cria **tabelas** com campos configuráveis (texto curto/longo,
  data, dropdown, relacionamento, arquivo, usuário, grupo de campos, entre
  outros).
- Cada tabela pode ter **9 estilos de visualização**: lista, galeria,
  documento, cartão, mosaico, kanban, fórum, calendário e gantt.
- Tabelas podem ter **scripts de usuário** (`beforeSave`, `afterSave`,
  `onLoad`) executados em sandbox isolado no servidor, permitindo lógica de
  negócio sem precisar de deploy.
- **Relacionamentos** entre tabelas (1:1, 1:N, N:N) são modelados pela UI e
  materializados como referência (Mongo) ou chave estrangeira real (Postgres).
- **Menus** e **páginas customizadas** organizam a navegação do app final.
- **Dashboards** compõem gráficos e cartões de métrica sobre os dados das
  tabelas.
- **RBAC** com grupos customizáveis controla quem vê e edita o quê, até o
  nível de campo e linha.
- Múltiplos **ambientes** (workspaces) isolados podem coexistir na mesma
  instalação, cada um com seu próprio conjunto de tabelas, usuários e dados.

## Arquitetura em uma frase

Backend Fastify + TypeScript fala direto com o driver do MongoDB (sem ODM); o
schema de cada tabela do low-code (`_schema`) é interpretado em runtime por um
`RowStore`, que grava os dados numa collection Mongo ou — se a instalação
escolheu assim no Setup Wizard — numa tabela Postgres real, com coluna tipada
por campo. O frontend é um SPA com SSR (TanStack Start) que consome a API REST
e um canal Socket.IO para notificações e chat.

## Casos de uso típicos

- CRM / ERP internos montados sem escrever backend.
- Formulários e fluxos de aprovação com automações via script.
- Portais internos com controle de acesso granular por grupo/linha/campo.
- Catálogos e bases de conhecimento publicados como tabelas públicas.
- Extensão da plataforma com plugins/módulos/ferramentas próprias quando o
  low-code não cobre um caso específico.

## Próximos passos

- [Instalação](installation.md) para subir a plataforma localmente ou em um
  servidor.
- [Primeiros Passos](quickstart.md) para os comandos de desenvolvimento do
  dia a dia.
- [Arquitetura](../architecture/overview.md) para entender a estrutura do
  monorepo e dos serviços.
