# Automações

LowCodeJS oferece dois mecanismos de automação: scripts de usuário por
tabela (sempre disponível) e um modelador de processos BPM (extensão
opcional).

## Scripts de tabela (`beforeSave` / `afterSave` / `onLoad`)

O mecanismo central de automação é o script de usuário, configurado por
tabela e executado em sandbox isolado no servidor. Veja o detalhe completo em
[Tabelas Dinâmicas — Scripts de usuário](dynamic-tables.md#scripts-de-usuário-automação-por-tabela):
APIs disponíveis (`field`, `context`, `email`, `users`, `notify`, `utils`,
`console`), limites (heap de 128 MB, timeout de 5s) e o contrato de retorno.

## BPM Modeler (extensão `core/tools/bpm-modeler`)

Ferramenta que embute o editor visual **bpmn-js** (bpmn.io) para desenhar
processos BPMN 2.0 e, num passo de mapeamento, gerar tabelas e os "métodos da
tabela" (código em `beforeSave`/`afterSave`) a partir do diagrama.

### Conceitos

- Um **processo** é persistido com o XML BPMN inteiro e uma lista de
  **elementos** mapeáveis (`userTask`, `serviceTask`, `sendTask`,
  `businessRuleTask`, `exclusiveGateway`).
- Mapear um elemento gera um **snippet** de código entre marcadores
  (`// @bpmn:<id> start` / `// @bpmn:<id> end`) dentro do método da tabela —
  remapear substitui só o bloco, remover apaga só o bloco, e edição manual
  fora do bloco sobrevive.
- `userTask` → cria/edita uma tabela, com campos derivados de um schema
  **form-js** (editor de formulário visual do bpmn.io): `textfield→TEXT_SHORT`,
  `textarea→TEXT_LONG`, `select/radio/checklist→DROPDOWN`,
  `datetime→DATE`, `number→TEXT_SHORT (INTEGER/DECIMAL)`,
  `checkbox→DROPDOWN Sim/Não`.
- `serviceTask`/`sendTask` → gera chamada a `email`/`notify`/código
  customizado no `afterSave` da tabela de origem (a `userTask` mais próxima a
  montante no fluxo).
- `businessRuleTask` → editor de tabela de decisão **dmn-js**; um compilador
  próprio traduz regras simples (hit policy FIRST/UNIQUE, comparações FEEL
  básicas) para `if` no `beforeSave`. Regras fora do subconjunto coberto
  deixam um `// TODO` no código gerado.

### Capacidade e acesso

Exige a capacidade de área `USE_BPM_MODELER` (não `MANAGE_TOOLS` — a tool
cria tabelas e grava código executado no servidor, então tem capacidade
própria) mais `ExtensionActiveMiddleware` ligado no ambiente.

### Estado

O modelador está implementado (CRUD de processo, parser/diff/apply via
`bpmn-moddle`, editor bpmn-js, tela de mapeamento, integração com IA para
sugestão de mapeamento, dmn-js e form-js). Detalhe de fases e decisões de
desenho em `_doc-bpm-modeler.md`, na raiz do repositório principal.
