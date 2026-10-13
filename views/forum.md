# Fórum

A visualização de Fórum organiza os registros como **canais** de discussão,
com mensagens e respostas — pensada para espaços de conversa em equipe,
suporte ou discussão de um assunto.

## Estrutura

Cada registro da tabela funciona como um canal, e dentro dele existe uma
lista de mensagens modelada como um [grupo de campos](../tables/field-groups.md),
com autor, texto e data de cada mensagem.

## Menções e notificações

Dentro de uma mensagem, mencionar outro usuário (`@usuário`) dispara uma
notificação em tempo real para a pessoa mencionada — o mesmo mecanismo usado
nos comentários do [Kanban](kanban.md).

## Composição de mensagem

O campo de texto das mensagens aceita o mesmo editor de texto rico usado em
outros pontos da plataforma (negrito, listas, links), o que permite mensagens
mais elaboradas, não só texto simples.

## Começando com um modelo pronto

A forma recomendada de montar um Fórum é usar o modelo pronto pela
ferramenta [Clonar Tabela](../tools/clone-table.md), que já cria a estrutura
de canais e mensagens corretamente — incluindo os campos exatos que a
plataforma espera para reconhecer a tabela como um Fórum.

## Pré-requisitos para habilitar manualmente

Montando do zero, a visualização de Fórum exige um grupo de campos de
mensagens com, no mínimo, identificador da mensagem, texto, autor e data, e
um campo correspondente na tabela principal apontando para esse grupo.

## Próximos passos

- [Kanban](kanban.md) — outra visualização que também usa comentários com
  menção.
- [Grupos de campos](../tables/field-groups.md)
