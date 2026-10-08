# Modelo de Ameaças

Resumo estruturado de `lowcodejs-threat-model.md` (repositório principal),
derivado de uma varredura de 1828 arquivos `.ts`/`.tsx` de `backend/` e
`frontend/`. Todo caminho de abuso no documento original está ancorado em
`arquivo:linha`; aqui fica só o resumo navegável.

## O sistema

Plataforma low-code self-hostable. O usuário configura tabelas pela UI e o
backend monta o `RowStore` da tabela **em runtime** a partir dessa
configuração, numa collection Mongo ou numa tabela Postgres (conforme a
escolha do Setup Wizard). Um campo dessa configuração,
`methods.{onLoad,beforeSave,afterSave}.code`, é **código JavaScript executado
no servidor** a cada gravação de linha — a superfície de risco mais sensível
do sistema.

Multi-tenant por **workspace**: o escopo é um `AsyncLocalStorage` fixado num
hook `onRequest` a partir do header `X-Workspace`; rows de cada workspace
vivem num database próprio no Mongo ou num schema próprio no Postgres.

## Ambiente e exposição

Instância pensada para rodar na internet aberta, com **sign-up livre**.
Deploy por Docker Compose ou Coolify. Um processo Node, um MongoDB (um
`MongoClient`, dois databases), um Postgres opcional, um Redis compartilhado
por toda a instalação.

## Ativos

| Ativo | Onde vive |
| --- | --- |
| Chave privada JWT, `COOKIE_SECRET` | `process.env` do container da API |
| Credencial S3, senha SMTP, chave OpenAI/LLM, token MCP | documento `Setting`, cifrado em repouso (ver [ADR 0009](../architecture/decisions.md)) |
| String de conexão de origens de importação | texto puro — não coberto pela ADR 0009 |
| Cofre de senhas do módulo `senhas` | mesmo serviço de cifragem do Setting |
| Linhas das tabelas de cada tenant | database/schema por workspace |
| Arquivos enviados | `_storage/` local ou bucket S3 |
| Log de auditoria | coleção `logs`, com UI de leitura em `/logs` |

## Atores e capacidades

| Ator | Capacidades | Não tem |
| --- | --- | --- |
| Anônimo | Rotas públicas (`/`, `/health-check`, `/setting/public`, `/setup/*`, autenticação, catálogo, upload de parte, tabelas com auth opcional) | Sessão |
| `REGISTERED` | Cadastro livre; cria tabela e vira `OWNER` dela; envia arquivo; menciona usuários | Nenhuma capacidade de área |
| `MANAGER` | Capacidades de área que o grupo conceder | Autoridade de instalação |
| `ADMINISTRATOR` | `isPrivileged`: bypassa o guard de linha, lê `/logs` | Não é MASTER, por desenho |
| `MASTER` | Super admin da instalação | — |
| Operador de outro tenant | Membro de outro workspace | Membership no workspace da vítima |

**Não-capacidades assumidas** (para não inflar severidade): o atacante não
tem acesso ao host, ao Mongo, ao Redis nem às variáveis de ambiente; não
intercepta TLS; não controla o registro npm nem o Docker Hub.

## Fronteiras de confiança

1. **Rede → API**: 241 handlers de rota declarados por decorator, contrato
   versionado em `schema.generated.ts`, guarda de cada rota conferida contra
   um baseline de autorização (`pnpm authz:check`).
2. **Requisição → plano de tenant**: `X-Workspace` fixa o escopo **antes** da
   autenticação; a participação só é conferida dentro do middleware de
   autenticação (e na auth opcional, nem roda).
3. **Configuração de tabela → runtime**: slug de campo vira path do schema;
   `methods.*.code` vira código executado.
4. **Guest do sandbox → processo**: o script roda num isolate do
   `isolated-vm`, com heap (128 MB) e realm próprios — builtins do host não
   entram mais no sandbox. O que resta é a autoridade de cada ponte exposta
   (`email`, `users`, `notify`, `field`).
5. **API → serviços externos**: destinos configuráveis (S3, LLM, transcrição,
   repositório, Mongo de origem de import) passam por uma guarda de SSRF —
   DNS resolvido e conferido antes de conectar, IP interno recusado,
   redirecionamento revalidado.
6. **Servidor → navegador**: SSR com desidratação do QueryClient; CSP em modo
   report-only (ainda não bloqueante).
7. **Conteúdo de usuário → DOM**: o campo de texto rico guarda markdown com
   ilhas de HTML cru (necessárias para imagem formatada e menção), então o
   viewer roda `rehypeRaw` seguido de sanitização por allowlist fechada de
   tags/atributos/estilos.

## Caminhos de abuso priorizados (reconferidos em 2026-09-24)

| Caminho | Impacto | Status |
| --- | --- | --- |
| Sign-up → cria tabela → grava `beforeSave` → RCE no container | Total | Fechado pela troca de `node:vm` por `isolated-vm` |
| Sign-up com nome `a"><script>` → menção → XSS armazenado | Sessão de qualquer leitor | Fechado (escape de `id`/`label` + sanitização pós-`rehypeRaw`) |
| Deploy novo → `/setup/step/email` → SMTP do atacante → captura de recuperação | Total e persistente | Fechado (segredo de bootstrap obrigatório em produção) |
| Anônimo → força bruta no código de recuperação | Conta | Baixa (8 dígitos, 5 tentativas/10min com bloqueio de 20min, cota por IP+email) |
| `MANAGE_USER_GROUPS` → auto-concessão → `MANAGE_USERS` → senha do MASTER | Instalação | Média |
| `REGISTERED` com `staticName: "../"` → escrita em `extensions/` | Total | Fechado (allowlist fechada de `STATIC_KEY_NAMES`) |
| `REGISTERED` sem header → lista usuários de todos os tenants | Confidencialidade cross-tenant | Fechado (rota exige `MANAGE_USERS`) |
| `REGISTERED` → canal não-privado do cofre `senhas` → segredos decifrados | Cofre de todos os tenants | Reduzido (capacidade `USE_PASSWORD_VAULT`) |
| `MANAGE_TOOLS` → Mongo da própria instalação → dry-run sobre `users` | Hashes e credenciais | Média |
| `CREATE_FIELD` → relacionamento forjado → leitura/escrita em qualquer collection | Dados do workspace | Média |
| `MANAGE_TOOLS` → desliga `core:row-access` → lê toda linha sigilosa | Dados do workspace | Média |
| XSS em sessão de admin → credenciais no DOM de `/settings` | Instalação | Média |

## Mitigações existentes que funcionam

Registradas para não serem removidas por engano: `workspace-scope.core.ts`
falha fechado (lança quando falta plano); `ScopedCollection` injeta o filtro
do plano em toda operação dos models `scoped`; JWT verificado com
`algorithms: ['RS256']` explícito (fecha confusão de algoritmo); bcrypt custo
12; `sessionVersion` revoga sessão; `query-builder` usa allowlist de campo;
`SearchService` escapa metacaractere de regex; guarda de zip-bomb no reader de
arquivo; ETag conferido em cada parte de upload; allowlist em
`GET /setting/public`; comparação com `timingSafeEqual` na URL assinada;
`socket-auth.protect` recarrega o usuário no connect; extensões do frontend
carregam por `import.meta.glob` estático, sem import remoto.

## Fora deste modelo

CI/CD, build e suíte de teste são tratados à parte (ver
[Hardening](hardening.md)). Comprometimento do host, do Mongo, do Redis ou do
registro de pacotes está fora do escopo.
