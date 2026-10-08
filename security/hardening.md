# Hardening

Resumo da auditoria completa de segurança e da correção que se seguiu.
Fontes: `security_best_practices_report.md`, `SECURITY.md` e
`.specs/features/security-hardening/` (spec, design, tasks), todos na raiz do
repositório principal.

## Método da auditoria

Varredura **arquivo a arquivo** dos 1828 arquivos `.ts`/`.tsx` do repositório,
em 12 fatias (backend `features/`, `services/`, `extensions/`, `core/`+
`model/`+`middlewares/`, infraestrutura; frontend `routes/_private/tables/`,
demais `routes/`, `components/`, `lib/`+`hooks/`+`integrations/`+
`extensions/`). Cada achado foi ancorado em `arquivo:linha` e cruzado com a
documentação oficial de cada peça da stack. Modelo de ameaça: instância na
internet aberta, multi-tenant, sign-up livre — detalhe completo em
[Modelo de Ameaças](threat-model.md).

## Resultado: 23 críticos, 3 padrões estruturais

A auditoria encontrou **23 achados críticos**: três vias independentes de
execução remota de código, quatro caminhos de escalada até administração,
cinco quebras de isolamento entre tenants, uma janela de sequestro da
instalação recém-implantada, e um XSS armazenado cujo payload é o próprio
nome da conta. A lista completa, com localização de código, está em
[Modelo de Ameaças](threat-model.md).

Mais relevante que a contagem, três **padrões estruturais** apareciam em
camadas independentes:

1. **Default-open na autorização de linha e de extensão** — o oposto do
   padrão fail-closed que o próprio repositório já usava para escopo de
   tenant.
2. **`MANAGE_TOOLS` era root de fato** — a mesma capacidade dava acesso a
   exportar tabela ignorando guard de linha, importar código
   server-side, clonar dados de outro tenant, conectar numa Mongo arbitrária
   e desligar a guarda de linha.
3. **Escopo de tenant aplicado por adesão, não por construção** — modelos de
   extensão que simplesmente não chamavam o helper de escopo viravam dados
   globais da instalação (entre eles, o cofre de senhas).

## O que a correção mudou

### Execução remota de código

O motor de sandbox trocou `node:vm` por `isolated-vm`: isolate próprio por
execução, builtins vindos do realm do isolate (nunca do host), APIs expostas
como ponte serializada, limite de heap e timeout que alcança código
assíncrono. Antes, o sandbox injetava `Object`, `Array`, `Promise` etc. **do
realm do host** como globais do contexto — `Object.constructor` era
literalmente o `Function` do host.

Uma segunda via (escrita arbitrária de arquivo via `staticName` escolhido
pelo cliente) foi fechada contendo a chave de storage a um valor gerado no
servidor.

### Força bruta e bootstrap

Rate limit (`@fastify/rate-limit` sobre Redis) em sign-in, sign-up,
magic-link, etapas do wizard e upload de parte. Código de recuperação ganhou
contador de tentativas, invalidação após erro e entropia maior. Rotação de
refresh token com detecção de reuso. As rotas `/setup/step/*` passaram a
exigir um segredo de bootstrap antes da conclusão do wizard, e respondem 404
depois de concluído.

### Escalada de privilégio

Os quatro caminhos foram fechados: privilégio do **alvo** (não só do ator) é
checado em patch de usuário; atribuição de grupo passa pelo fecho transitivo,
não pelo slug direto; patch de grupo não concede capacidade que o próprio
ator não possui, nem edita grupo do qual é membro; `members[].profile` de
tabela é ignorado salvo para o dono corrente; operações em lote exigem
predicado de elegibilidade (dono, não-MASTER, etc.).

### Isolamento de tenant

Cinco models de extensão (cofre de senhas, config de transcrição, dropdown em
cascata, campos condicionais) passaram a declarar escopo de workspace, cada
um com uma migration que carimba o dado existente. O guard de linha deixou de
falhar aberto: hoje distingue "nunca configurado" de "explicitamente
liberado para todas as tabelas", e qualquer `deny` de qualquer guard vence
(ver [ADR 0008](../architecture/decisions.md)).

`MANAGE_TOOLS` foi quebrado em capacidades por ferramenta
(`USE_CLONE_TABLE`, `USE_TABLE_IMPORT_EXPORT`, `USE_TEST_DATA_GENERATOR`,
`USE_INSTALLATION_IMPORT`, `USE_BPM_MODELER`), cada uma independente.

### Segredo e auditoria

O log de auditoria passou a gravar só uma allowlist de campos por rota —
nunca mais o corpo bruto (que incluía senha e outras credenciais). Os
segredos do `Setting` passaram a ser cifrados em repouso (ver
[ADR 0009](../architecture/decisions.md)) e a API nunca devolve o valor em
claro, nem para MASTER.

### XSS e headers

O viewer de texto rico passou a sanitizar o HTML depois do `rehypeRaw`, com
uma allowlist fechada de tags/atributos/estilos. CSP, `X-Content-Type-Options`,
`X-Frame-Options`, `Referrer-Policy` e `Permissions-Policy` passaram a ser
emitidos em toda resposta.

### Infraestrutura e dependências

CI ganhou verificação de vulnerabilidade de dependência (`npm audit`),
workflows com `permissions:` declarado e actions referenciadas por SHA. Os
containers do backend rodam com usuário não-root.

## Checagens novas para impedir reintrodução

Duas checagens determinísticas entraram no `boot:check`/gates do backend:

- **Model novo sem declaração de escopo** — falha, a menos que o model conste
  de uma lista explícita de globais-por-desenho (`GLOBAL_BY_DESIGN`).
- **Rota nova sem capacidade declarada** — `authz:check` confere a guarda de
  cada rota contra um baseline (`core/openapi/authz-baseline.json`).

## Metodologia de execução

Seguida a regra registrada na [ADR 0003](../architecture/decisions.md): o
agente que implementa a correção **escreve** o teste de regressão; o
mantenedor **roda** a suíte (custo e efeito colateral de MongoDB real ficam
do lado de quem decide rodar). O gate automático de cada tarefa é estático:
`tsc --noEmit`, `openapi:check`, `di:check`, `scope:check`, `boot:check` e
lint.

O plano completo (82 requisitos rastreados, 13 fases, ~170 tarefas, todas
concluídas) está em `.specs/features/security-hardening/{spec,design,
tasks}.md`, na raiz do repositório principal.

## Política de divulgação

Veja `SECURITY.md` no repositório principal para como reportar uma
vulnerabilidade (security advisory privado no GitHub), o que está dentro e
fora do escopo de report, e as responsabilidades de quem instala (trocar
senhas padrão, usar proxy reverso com TLS, concluir o wizard imediatamente,
não expor Mongo/Redis na rede).

Enquanto a correção de qualquer novo achado não estiver concluída, trate uma
instalação pública com sign-up livre como potencialmente comprometível — é a
postura que o próprio `SECURITY.md` recomenda.
