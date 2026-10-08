# Arquitetura — Frontend

Plataforma construída com React + TanStack Start (SSR) + TypeScript.

## Tech stack

| Tecnologia             | Versão          | Uso                          |
| ----------------------- | ---------------- | ------------------------------ |
| React                   | 19.2.0            | UI library                       |
| TanStack React Start    | ~1.168            | Meta-framework SSR (Nitro)        |
| TanStack React Router   | ~1.170            | File-based routing                |
| TanStack React Query    | 5.66.5            | Server state management           |
| TanStack React Table    | 9.2               | Headless table                     |
| React Hook Form         | 7.68.0            | Form management                   |
| @hookform/resolvers     | 5.2.2             | Ponte entre RHF e VineJS           |
| TanStack React Virtual  | 3.13.20           | Virtualização                     |
| Base UI                 | 1.x               | Primitivos acessíveis              |
| Tailwind CSS            | 4.0.6             | Utility-first CSS                  |
| CVA                     | 0.7.1             | Variantes de componentes            |
| Monaco Editor           | 4.7.0             | Editor de código                    |
| Tiptap                  | 3.31.3            | Editor WYSIWYG                     |
| @dnd-kit                | core + sortable   | Drag and drop                      |
| Socket.IO Client        | 4.8.3             | WebSocket (chat)                   |
| VineJS                  | 4.4.0             | Validação                          |
| Phosphor Icons          | 2.1.10            | Ícones                             |
| Vite                    | 8.3               | Build tool (Rolldown)              |
| Nitro                   | 3.0.260903-beta   | Servidor SSR do Start               |
| Vitest                  | 5.0               | Testes                              |

Start e Router são pinados com `~` de propósito: as majors/minor deles quebram
tipo entre si.

## Arquitetura

```
router.tsx → __root.tsx (Settings + Meta)
                ├── /_authentication (guard público) → Sign In, Sign Up
                └── /_private (guard autenticado) → Sidebar + Header
                        ├── dashboard
                        ├── tables (core do low-code)
                        ├── users, groups, menus, pages
                        ├── profile, settings, tools
                        └── demais áreas (16 ao todo)
```

## Fluxo de inicialização

1. `router.tsx` — `createRouter()` com `routeTree` (auto-gerado) + queryClient
   como contexto
2. `__root.tsx` — server function carrega settings do backend, meta tags
   OG/Twitter/structured data
3. `/_authentication/layout.tsx` — `beforeLoad`: se logado, redireciona para
   `DEFAULT_ROUTE`
4. `/_private/layout.tsx` — `beforeLoad`: `ensureQueryData` de contas e
   perfil, semeando o QueryClient; 401 limpa o cache e redireciona para `/`
   (exceto em tabela pública)

Preload strategy: `intent` (prefetch ao hover). Scroll restoration habilitado.

## Estrutura de diretórios

```
frontend/
├── src/
│   ├── router.tsx
│   ├── env.ts                      # Checagem de env vars a mão
│   ├── styles.css                  # Tailwind + custom styles + dark mode
│   ├── routeTree.gen.ts            # Auto-gerado
│   │
│   ├── routes/                     # File-based routing (430 arquivos)
│   │   ├── __root.tsx
│   │   ├── setup/                  # Setup Wizard
│   │   ├── _authentication/        # Rotas públicas
│   │   └── _private/               # 16 áreas protegidas
│   │       ├── dashboard/ tables/ users/ groups/ menus/ pages/
│   │       ├── profile/ settings/ tools/ extensions/
│   │       ├── e/                  # Telas de módulos de extensão
│   │       ├── workspaces/ w/      # Lista/CRUD e ativação de ambientes
│   │       └── logs/ error-logs/ notifications/
│   │
│   ├── components/
│   │   ├── ui/                     # Design system (37 componentes)
│   │   └── common/                 # Kit de negócio (22 subdiretorios)
│   │       └── dynamic-table/      # Campos tipados, config de campo
│   │
│   ├── hooks/                      # Custom React hooks
│   ├── lib/                        # Utilitários, enums, interfaces, validators
│   ├── start.ts                    # Middleware de request (escopo por requisição)
│   └── integrations/
│       ├── tanstack-query/         # http.ts, queries.ts, mutations.ts
│       └── react-hook-form/        # AppField + binding de campo
│
├── extensions/                     # Código UI das extensões
├── vite.config.ts
└── Dockerfile-*
```

## Cliente API

**Não há cliente HTTP dedicado.** `request<T>()`
(`integrations/tanstack-query/http.ts`) embrulha `fetch` com duas garantias:

1. **Resposta de erro vira exceção** — `fetch` só rejeita em falha de rede;
   401/422/500 chegam como promessa resolvida. `request` normaliza isso.
2. **Sessão expirada renova sem a tela perceber** — o 401 do guard dispara
   refresh e replay da requisição original, deduplicado por requisição no SSR
   via `AsyncLocalStorage`.

`send()` devolve a `Response` crua (usado pela exportação CSV, que precisa do
`Content-Disposition` e do corpo em `Blob`).

**Envelope de erro**: `{ message, status, code, errors? }`. `code` é o
identificador estável que a tela compara — nunca `message`.

**`X-Workspace`** entra no próprio `request()`, lido do cookie que o store da
aba espelha.

## Upload de arquivo

O binário não vai num `POST` para a API. `lib/multipart-upload.ts` (motor) +
`hooks/use-multipart-upload.ts` (casca React):

1. `POST /files` com metadados → URL assinada por parte
2. O navegador fatia com `Blob.slice()` e faz `PUT` em cada URL, direto no
   armazenamento (até 4 partes em voo, via XHR para expor progresso)
3. `POST /files/:id/complete` com os `ETag` coletados

O `fileId` fica em `localStorage`, chaveado por nome/tamanho/data, para
retomar upload interrompido. Cinco fluxos continuam enviando binário direto
para a API (chat, import CSV, import de tabela por planilha, transcrição de
documento, `.zip` do envio em lote) — todos porque o servidor precisa **ler**
o conteúdo.

## Variáveis de ambiente

| Variável          | Tipo | Default               | Descrição                                            |
| ------------------ | ---- | ----------------------- | ------------------------------------------------------- |
| VITE_API_BASE_URL  | url  | http://localhost:3000   | URL base da API, vista do navegador. Entra no bundle em build |
| SERVER_API_URL     | url  | (opcional)               | API vista do servidor de SSR. Runtime                    |
| SERVER_URL         | url  | (opcional)               | URL pública do próprio app. Runtime                       |

`VITE_*` é estático por desenho do Vite: mudar a URL da API exige rebuild da
imagem, não troca de variável no container.

## Estilização

- **Tailwind CSS v4** via `@tailwindcss/vite` plugin
- Cores custom em `oklch` (brand-orange, brand-blue-dark/mid), design tokens
  (`--radius`, `--font-sans/mono`), dark mode via classe `.dark`
- Cores semânticas (`success`, `warning`, `info`, `destructive`, `muted`,
  `primary`, `link`) — nunca paleta fixa fora de `components/ui/`
- `cn()` em `lib/utils.ts`: `clsx(...inputs) → twMerge()`

## Estado global

### Sessão: não há store

Quem está logado é dado do servidor, lido do cache do TanStack Query.
`hooks/use-session.ts` usa as mesmas factories que o `beforeLoad` de
`_private`, então a chave bate byte a byte.

### Ambiente ativo: não há store

A lista de ambientes é dado do servidor (cache). Qual deles está ativo é uma
string que vive num cookie (`lib/workspace-cookie.ts`) — necessário para SSR,
já que é o único canal que o navegador manda junto do documento.

### TanStack Query

`staleTime` de 1 minuto, `retry` sem insistir em 4xx no navegador, zero
retry no servidor (SSR), uma instância de `QueryClient` por request no SSR.
Só invalidação — zero `setQueryData`, zero update otimista.

## Roteamento

File-based routing com TanStack Router. Rotas auto-geradas em
`routeTree.gen.ts`.

| Pattern          | Significado                                          | Exemplo                        |
| ---------------- | ------------------------------------------------------ | --------------------------------- |
| `index.tsx`      | Route config (`createFileRoute`) com loader/beforeLoad | `tables/index.tsx`                 |
| `index.lazy.tsx` | Stub de rota: só `component:`                           | `tables/index.lazy.tsx`            |
| `$param`         | Segmento dinâmico                                        | `$slug/`, `$userId/`               |
| `-components/`   | A tela e as peças dela (ignorada pelo roteador)          | `tables/-components/table.tsx`      |
| `_prefixed/`     | Pathless layout (não gera URL)                            | `_authentication/`                  |
| `layout.tsx`     | Layout wrapper para rotas filhas                          | `_private/layout.tsx`               |

Convenção: rota é só rota, a tela desce para `-components/`, e o componente lê
a rota por `getRouteApi`.

## Sistema de permissões (RBAC)

Espelha o backend: grupos custom + capacidades de área + bindings por ação.
Veja [RBAC](../features/rbac.md). No frontend:

- `lib/menu/menu-access-permissions.ts`: `canAccessRoute(...)`, `DEFAULT_ROUTE`
- `lib/menu/menu.ts`: monta os menus a partir das capacidades
- `hooks/use-table-permission.ts`: permissões granulares (VIEW/CREATE/UPDATE/
  REMOVE), binding-aware

O backend é sempre a fonte de verdade do enforcement — o frontend decide só o
que mostrar.

## Formulários

Sistema central em `integrations/react-hook-form/`:

- `field-context.tsx`: `AppField` — um `Controller` que publica
  `field`/`fieldState` num contexto
- `use-field-validation.ts`: binding do campo mais `{ isInvalid, errors }`

O formulário usa `useForm` com `resolver: vineResolver(XValidator)`, validator
sempre vindo de `lib/validator.ts`. Formulários **dinâmicos** (campos montados
em runtime a partir de `IField[]`) não têm schema: usam `rules` por campo com
`buildFieldValidator`.

### Field components (4 grupos)

| Grupo          | Diretório                                             | Exemplos                                            |
| -------------- | ------------------------------------------------------- | ------------------------------------------------------ |
| Base (leves)   | `components/common/dynamic-table/base/`                 | FieldText, FieldEmail, FieldSwitch, FieldFileUpload       |
| Rich (pesados) | `components/common/dynamic-table/rich/`                 | FieldEditor (Tiptap)                                      |
| Table Config   | `components/common/dynamic-table/table-config/`         | TableFieldTypeSelect, TableFieldDropdownOptions            |
| Table Row      | `routes/_private/tables/$slug/-components/table-row/`   | TableRowTextField, TableRowDateField, TableRowRelationshipField |

Monaco e o Tiptap entram por `React.lazy`.

## Componentes UI (Design System)

37 componentes em `components/ui/` baseados em Base UI + shadcn pattern:

- CVA para variantes de estilo
- `data-slot` attribute em todo componente raiz
- Composição pela prop `render` do Base UI (`useRender` + `mergeProps`) — não
  existe `asChild`
- Compound components para componentes complexos (Card, Table, Sidebar, Dialog)

## Componentes de negócio

22 subdiretórios em `components/common/`. O mais importante é
`dynamic-table/` (4 subdirs: `base/`, `rich/`, `table-config/`,
`table-selectors/`).

## Build & Deploy

| Comando       | Descrição                                               |
| -------------- | ---------------------------------------------------------- |
| `pnpm dev`     | Vite dev server (SSR hot reload, porta 5173)                 |
| `pnpm build`   | Build produção                                                |
| `pnpm start`   | Serve `.output/server/index.mjs` (Nitro)                       |
| `pnpm test`    | Vitest                                                         |
| `pnpm lint`    | Prettier + ESLint (auto-fix)                                   |

### Docker

| Dockerfile            | Uso                                              | Porta |
| ----------------------- | --------------------------------------------------- | ----- |
| Dockerfile-local        | Desenvolvimento (`pnpm dev --host`)                    | 5173   |
| Dockerfile-production   | Copia o `.output` pronto, usuário non-root              | 3000   |
| Dockerfile-coolify      | Multi-stage, usuário non-root, curl para healthcheck    | 3000   |

Logos, nome do sistema e descrição vivem no documento `Setting` do MongoDB e
são carregados em runtime via server function — não são build args.

## Extensões

Espelho frontend de `backend/extensions/` para o código de UI. Workshop de
gestão em `/extensions` (rota MASTER): lista as extensões registradas,
toggle de ativação, configuração de escopo por tabela. Veja
[Extensões](../features/extensions.md).
