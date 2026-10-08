# Extensões

Mecanismo build-time + ativação runtime para estender a plataforma sem mexer
no core. Diretório canônico: `backend/extensions/`. O frontend espelha só o
código de UI em `frontend/extensions/`.

## Tipos de extensão

| Tipo | Descrição | Exemplo | Onde aparece |
| --- | --- | --- | --- |
| `PLUGIN` | Botão/widget pequeno injetado num placeholder do core | "Exportar PDF" | Slot configurado no manifest (ex: `table.actions`) |
| `MODULE` | Tela/dashboard/formulário maior, com URL própria | Dashboard customizado | `/e/<pkg>/<id>` ou via Menu custom |
| `TOOL` | Utilitário do sistema, listado no submenu Ferramentas | Clonar tabela, Importar CSV | `/tools/<pkg>/<id>` + sublink na sidebar |

## Pacotes

| Pacote | Ativação | Conteúdo |
| --- | --- | --- |
| `core` | auto-ativado no boot | tools/plugins/modules shipados com a plataforma |
| `apps` | opcional (começa desativado) | extensões oficiais opcionais (ex: dashboard, senhas) |
| `forms` | opcional (começa desativado) | plugins que enriquecem os formulários das tabelas |

## Estrutura de pastas

```
backend/extensions/
└── <pkg>/
    ├── manifest.json
    ├── plugins/<id>/manifest.json   # obrigatório
    ├── modules/<id>/manifest.json   # obrigatório
    └── tools/<id>/manifest.json     # obrigatório
```

A chave única no DB é o trio `(pkg, type, extensionId)` — `pkg` é a pasta
direta sob `extensions/`, `type` deriva da pasta intermediária
(`plugins`→`PLUGIN`, `modules`→`MODULE`, `tools`→`TOOL`), `extensionId` é a
pasta da extensão e precisa bater com `manifest.id`.

## `manifest.json`

Validado por VineJS (`application/services/extension/manifest.schema.ts`).
Exemplo mínimo para um `TOOL`:

```json
{
  "id": "clone-table",
  "type": "TOOL",
  "name": "Clonar Tabela",
  "version": "1.0.0",
  "icon": "CopyIcon",
  "tool": { "submenu": "tables" }
}
```

Campos comuns: `id`, `type`, `name`, `description`, `version`, `author`,
`icon` (nome do `@phosphor-icons/react`, cai em `WrenchIcon` se não estiver na
allowlist), `image`, `requires.lowcodejs` (versão mínima), `requires.extensions`,
`permissions.view` (roles permitidas; vazio = todos os usuários
autenticados).

## Ciclo de vida

- O **loader** (`application/services/extension/`) varre `extensions/` no
  boot, valida cada manifest e faz upsert na collection `extensions`.
- Extensão com `boot.ts` tem o arquivo importado no mesmo passo. O que precisa
  do servidor de pé (namespace de socket, worker de fila) é agendado por
  `ExtensionRuntimeContractService.onServerStart` e disparado por
  `bin/server.ts`.
- **Regra de dependência**: nada em `application/`, `hooks/`, `bin/` ou
  `start/` importa de `extensions/`. A direção é sempre extensão → core.
- Dois models: `Extension` (catálogo, DB de sistema, chave única
  `(pkg, type, extensionId)`, flags `enabled`/`available`) e
  `ExtensionWorkspaceState` (`scoped: true`) — a mesma extensão pode estar
  ligada num ambiente e desligada noutro.
- `ExtensionActiveMiddleware({ pkg, type, extensionId })` retorna 404
  (`EXTENSION_NOT_ACTIVE`) quando a extensão está desativada no ambiente da
  request ou indisponível.
- **Sem sandbox**: extensões rodam com privilégios totais — desenvolvedores
  internos assumem o risco. Isso é por que capacidades específicas por
  ferramenta (`USE_CLONE_TABLE`, etc.) existem em vez de um `MANAGE_TOOLS`
  genérico cobrindo tudo.

## REST de gestão

`/extensions` — list e toggle exigem `MANAGE_TOOLS`; configurar escopo por
tabela exige `MANAGE_PLUGINS`; `GET /extensions/active` só exige sessão.

## Gerador de boilerplate

```bash
pnpm make:extension                                   # interativo
pnpm make:extension --type plugin --id hello-table
pnpm make:extension --type tool --pkg core --id merge-tables --with-backend
```

Gera a estrutura da extensão nos dois lados do monorepo (backend canônico +
frontend UI) seguindo as convenções do loader. Flags relevantes: `--type`,
`--pkg` (default `core`), `--id`, `--name`, `--description`, `--icon`,
`--slots` (para `PLUGIN`), `--submenu` (para `TOOL`), `--roles`,
`--with-backend`/`--no-backend`, `--force`, `--yes`.

Script fonte: `scripts/make-extension.mjs`.

## Extensões existentes

Ver [Mapa de Funcionalidades](../architecture/feature-map.md#extensões) para
a lista completa com rotas e o que cada uma toca.
