# RBAC

O modelo de permissões foi reescrito: não há mais 4 roles fixos governando
tudo. O controle real gira em torno de **grupos custom + capacidades de área +
bindings por ação**. Os 4 roles legados seguem existindo só para
compatibilidade (JWT, derivação).

## Grupos custom com hierarquia

- Grupos são configuráveis e podem **englobar** outros via `encompasses[]`
  (fecho transitivo): um grupo herda as permissões de tudo que engloba.
- Um usuário pertence a **vários grupos** (`user.groups[]`). O campo legado
  `user.group` (singular) foi mantido para compat e fallback.
- Resolvido por `application/services/user-group/`:
  `resolveUserGroupIds(user)` (fecho dos ids) e `resolveCapabilities(user)`
  (união das permissões do fecho).

## Capacidades de área (`E_AREA_CAPABILITY`)

14 capacidades no backend, atribuíveis a qualquer grupo:

- `MANAGE_USERS`, `MANAGE_MENU`, `MANAGE_USER_GROUPS`, `MANAGE_SETTINGS`,
  `MANAGE_TOOLS`, `MANAGE_PLUGINS`, `MANAGE_CHAT`, `MANAGE_WORKSPACE`
- Uma por ferramenta: `USE_CLONE_TABLE`, `USE_TABLE_IMPORT_EXPORT`,
  `USE_TEST_DATA_GENERATOR`, `USE_INSTALLATION_IMPORT`, `USE_BPM_MODELER`
- `USE_PASSWORD_VAULT` (o cofre de senhas, módulo `senhas`)

Enforçadas por `PermissionMiddleware(capability)`, substituindo o
`RoleMiddleware` nas áreas do sistema. MASTER sempre passa (bypass). O
frontend (`lib/constant.ts`) espelha só as 8 primeiras (`MANAGE_*`) — as seis
`USE_*` não têm entrada na constante do frontend ainda.

## Bindings por ação (`E_PERMISSION_TARGET`)

Alvo `{ kind, group }` onde `kind` é `PUBLIC` (inclui visitante), `NOBODY` ou
`GROUP`. Para `GROUP`, o acesso é por **interseção**: só libera se o grupo
estiver no fecho do usuário **e** o fecho de capacidades contiver a permissão
global da ação. Dono, membros da tabela e `PUBLIC` não passam pela
interseção.

Reusado em três lugares:

- **`table.permissions`**: mapa das 10 ações de tabela (viewTable,
  updateTable, createField, updateField, removeField, viewField, createRow,
  updateRow, removeRow, viewRow) → binding.
- **`field.permissions: { list, form, detail }`**: binding por contexto.
  Enforçado server-side (`FieldContractService`) — remove valores de campos
  ocultos nas respostas de linha e descarta escritas em campos ocultos.
  Campos nativos e usuários privilegiados nunca são filtrados. Ausência de
  binding para um contexto significa campo **oculto**.
- **`menu.visibility`** → binding de visibilidade do item de menu.

## Membros da tabela

`table.members[]` (`{ user, profile }`) com perfis fixos
(`E_TABLE_PROFILE`): `OWNER`, `ADMIN`, `EDITOR`, `CONTRIBUTOR`, `VIEWER`,
avaliados pela matriz `TABLE_PROFILE_MATRIX`. `CONTRIBUTOR` edita/remove
apenas as próprias linhas (`OWN`). `OWNER` tem acesso total e pode trocar o
dono.

## Sem fallback legado

Os campos antigos `visibility`/`collaboration`/`administrators` (tabela) e
`showInList`/`showInForm`/`showInDetail` (campo) foram **removidos** do
schema, dos tipos e dos enums. O enforcement lê **somente**
`table.permissions`/`table.members`/`table.owner` e `field.permissions`.
Tabelas novas nascem no modelo novo (preset `RESTRICTED`, dono como membro
`OWNER`) — nunca com `permissions: null`.

## Guard de linha (opcional, via extensão)

Além dos bindings acima, uma extensão (`core/plugins/row-access`) permite
configurar ACL por linha. Decisão registrada nas ADRs 0002 e 0008: um guard
que responde `abstain` conta como "permitir" (não é opinião sobre aquela
linha), mas qualquer `deny` de qualquer guard **vence** — os guards se
combinam em AND, nunca em OR. Veja [Decisões de Arquitetura](../architecture/decisions.md).

## JWT

Payload inalterado: `{ sub, email, role, type }`. `role` ainda é derivado do
grupo **principal** (`user.group`, singular) só para compat — **não é usado
para autorizar**. O privilégio real (acesso total MASTER/ADMINISTRATOR) é
resolvido pelo fecho de grupos via `UserGroupContractService.isPrivileged(user)`
a cada request, servidor-side.

## No frontend

O backend é sempre a fonte de verdade do enforcement; o frontend só decide o
que mostrar:

- `lib/menu/menu-access-permissions.ts`: rotas permitidas resolvidas pelas
  capacidades do usuário, `canAccessRoute(...)`, `DEFAULT_ROUTE`
- `lib/menu/menu.ts`: monta os menus a partir das capacidades
- `hooks/use-table-permission.ts`: permissões granulares (VIEW/CREATE/UPDATE/
  REMOVE para TABLE/FIELD/ROW), binding-aware
