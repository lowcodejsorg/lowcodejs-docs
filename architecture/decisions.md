# Decisões de Arquitetura

Resumo das ADRs (Architecture Decision Records) do repositório principal,
localizadas em `docs/adr/`. Cada uma registra uma decisão que vale contexto —
inclusive risco aceito — para quem for mexer na área depois.

## ADR 0001 — Segredo do Setting fica em texto puro no Mongo

**Status: substituída pela ADR 0009.** Decisão original: os segredos do
documento `Setting` (chave S3, senha SMTP, chave OpenAI...) ficariam em texto
puro, protegidos por `select: false` e allowlist de resposta, não por
criptografia. O argumento caiu quando se reconheceu que dump de backup,
réplica de leitura e snapshot de volume não carregam o `process.env` — então
o texto puro era entregue inteiro nesses caminhos.

## ADR 0002 — `abstain` continua significando "permitir" no guard de linha

Em `row-access-guard.service.ts`, um guard que responde `abstain` não
bloqueia. É o comportamento correto para toda tabela sem ACL de linha — negar
por ausência tornaria ilegível a maioria das tabelas. O defeito real (corrigido
à parte) era outro: extensão desligada apagava a ACL de uma tabela protegida,
em silêncio.

## ADR 0003 — O agente escreve o teste; quem roda é o humano

Regra do repositório: o gate automático de cada tarefa é estático (`tsc`,
`openapi:check`, `di:check`, `scope:check`, `boot:check`, lint). As suítes
(`test:e2e`, `test:unit`, vitest do frontend) são escritas, mas rodadas pelo
mantenedor — o custo e o efeito colateral (MongoDB real, timeout de 60s) são
dele.

## ADR 0004 — `ValidationToken` fica global, sem escopo de ambiente

`ValidationToken` é `scoped: false` por desenho, não por descuido: o token
nasce e morre antes de existir sessão (rotas públicas de recuperação/magic
link), então não há `X-Workspace` a considerar. Está registrado em
`GLOBAL_BY_DESIGN` em `scripts/scope-check.ts`.

## ADR 0005 — Cache de metadado de storage segue global por processo

`StorageMetaCacheService` continua com um mapa único por processo, chaveado
só por `filename`. Isso só é seguro porque a chave hoje é 16 bytes aleatórios
do servidor (depois de uma correção de segurança) mais o `storagePrefix` do
ambiente — nunca mais escolhida pelo cliente. Se algum caminho novo voltar a
deixar o cliente escolher a chave, esta ADR cai e o cache precisa ser
escopado por ambiente.

## ADR 0006 — Driver `mongodb` sem ODM: definição + store no lugar do Mongoose

O backend fala com o Mongo pelo driver oficial (`@fastify/mongodb`), sem
Mongoose. No lugar do schema: uma **definição** (`defineModel`) com a forma
gravada, `scoped`, `softDelete`, `hidden`, índices e virtual; e um **store**
(`ModelStore<T>`) com a `ScopedCollection` e a serialização para o domínio.
O store **não é repositório** — não tem método de consulta por caso de uso,
só a collection escopada, exatamente como a régua de arquitetura do backend
(`PADRAO-BACKEND.md` §1.2) exige. `_id` é `ObjectId` no banco e `string` na
fronteira de domínio. Populate usa loaders (`$in`), não `$lookup` — porque o
outro lado de uma ref pode estar em outro database/plano e precisa de
projeção própria.

## ADR 0007 — Banco das tabelas de dados: Mongo ou Postgres, escolhido uma vez

As tabelas de dados (tables × fields × rows) podem viver no MongoDB ou no
PostgreSQL; o sistema vive sempre no MongoDB. A escolha é da instalação, feita
uma vez no Setup Wizard, gravada no Setting de instância e **imutável por
construção** (o prefixo `DATA_` é instance-only, então `PUT /setting` a
descarta). Não há migração entre os dois bancos. Detalhe completo em
[Camada de Dados](data-layer.md).

## ADR 0008 — `deny` de qualquer guard vence no guard de linha

Substitui a regra de precedência "allow > deny" da ADR 0002 (o resto dela
segue valendo). Os guards de linha se combinam em AND: qualquer `deny` nega a
leitura; o primeiro `deny` é a resposta na escrita; todo fragmento de guard
entra no `$and` da listagem. Antes, um guard permissivo que respondesse
`allow` furava uma negação — decisão do dono do produto: "deny vence".

## ADR 0009 — Segredo do Setting é cifrado em repouso

Substitui a ADR 0001. Os seis segredos do `Setting` (chave/segredo S3, senha
SMTP, chave OpenAI, token MCP, chave LLM) passam a ser gravados cifrados, com
o mesmo serviço que já cifrava o cofre de senhas (`secret-crypto`, movido
para `application/services/`). Formato `enc:v2`: HKDF-SHA-256 (não hash cru),
AAD por campo (evita que um ciphertext copiado de um campo para outro decifre
no destino), `authTagLength: 16` explícito. `enc:v1` continua legível para
não quebrar cofres já gravados; escrita é sempre v2. A chave é a mesma
`PASSWORDS_ENCRYPTION_KEY` do cofre de senhas (com fallback para
`COOKIE_SECRET`) — nenhuma variável nova.

## ADR 0010 — O prefixo `I` das entidades (`IUser`, `ITable`...) fica

Decisão de **não** remover o prefixo `I` dos tipos de domínio, apesar de ele
vir de um tempo em que eram `interface` (hoje são `type`). Motivos: alcance
(286 arquivos no backend + 207 no frontend citam essas entidades), colisão de
nome (`Table` já é um componente no frontend) e ganho zero em comportamento.
Tipo novo de **entidade de domínio** segue o prefixo; payload/resposta/opções
não levam `I`.

## Onde estão as ADRs originais

Os arquivos completos, com a análise linha a linha e as referências de
código, estão em `docs/adr/0001-*.md` a `docs/adr/0010-*.md` no repositório
principal.
