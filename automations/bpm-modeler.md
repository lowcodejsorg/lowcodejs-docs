# Modelador de Processos (BPM)

O **BPM Modeler** é uma ferramenta (extensão opcional) que permite desenhar
um processo visualmente — como um fluxograma — e transformar cada etapa
desse desenho em tabelas e automações reais na plataforma, sem escrever
código manualmente.

## Como funciona, em linhas gerais

1. **Desenhe o processo** num editor visual de diagramas (o mesmo tipo de
   notação usada em fluxogramas de processo — BPMN), com etapas como "tarefa
   do usuário", "tarefa de serviço", "decisão" e "fim".
2. **Mapeie cada etapa** para o que ela deve gerar na plataforma: uma tarefa
   do usuário pode virar uma tabela nova (com um formulário próprio); uma
   etapa de serviço pode virar o disparo de um e-mail ou de uma notificação;
   uma decisão pode virar uma condição que direciona o fluxo.
3. **Aplique o mapeamento**: a ferramenta cria as tabelas necessárias e grava
   os [scripts de automação](scripts.md) correspondentes nelas.
4. Se o processo mudar depois (uma etapa for renomeada, removida ou uma nova
   for adicionada), a tela de mapeamento mostra o que está **novo**,
   **alterado** ou **ausente** em relação ao que já foi aplicado, para você
   decidir o que atualizar.

## Tipos de etapa suportados

| Etapa do diagrama | O que gera na plataforma |
| --- | --- |
| **Tarefa do usuário** | Uma tabela (com formulário próprio, que pode ser desenhado num editor de formulário visual) |
| **Tarefa de serviço** | Um e-mail, uma notificação interna, ou um ponto para lógica customizada |
| **Tarefa de envio** | Mesmas opções da tarefa de serviço |
| **Decisão (gateway)** | Uma condição que direciona o fluxo com base em um valor do registro |
| **Tarefa de regra de negócio** | Uma tabela de decisão (regras do tipo "se isso, então aquilo"), também editável visualmente |

## Assistente de IA no mapeamento

A ferramenta pode sugerir automaticamente como mapear cada etapa do diagrama,
usando o [Assistente de IA](../settings/ai-assistant.md) configurado na
instalação — uma forma de acelerar o mapeamento inicial, que depois pode ser
ajustado manualmente.

## Quando usar

Use o BPM Modeler quando o processo tem **várias etapas conectadas** e
você quer enxergar o fluxo inteiro visualmente antes de materializá-lo em
tabelas — para automações pontuais e simples, um [script](scripts.md) direto
numa única tabela costuma ser suficiente.

## Disponibilidade

O BPM Modeler é uma ferramenta opcional, ativada em **Extensões** pelo
usuário MASTER, e aparece no menu **Ferramentas** para quem tiver a
capacidade correspondente liberada no seu grupo.

## Próximos passos

- [Scripts](scripts.md) — o mecanismo de automação que o BPM Modeler usa por
  trás dos panos.
- [Visão geral das ferramentas](../tools/overview.md)
