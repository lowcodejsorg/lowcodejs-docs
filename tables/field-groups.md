# Grupos de campos

O campo do tipo **Grupo de campos** cria uma "subtabela" dentro do próprio
registro: um conjunto de campos que se repete quantas vezes for preciso,
sem precisar criar uma tabela relacionada separada.

## Quando usar

Use um grupo de campos quando a informação:

- Só faz sentido **dentro** do registro pai (não é algo que você precisaria
  listar, filtrar ou relacionar de forma independente).
- Se repete em quantidade variável dentro de cada registro.

Exemplos típicos:

- **Itens de um pedido** — cada pedido tem uma lista de itens, cada um com
  produto, quantidade e preço.
- **Tarefas de um quadro Kanban** — um cartão pode ter uma lista de
  subtarefas, cada uma com título e uma marcação de concluída.
- **Comentários** — uma lista de comentários associada ao registro, cada um
  com autor, texto e data.

## Grupo de campos x Relacionamento

A diferença principal em relação a um [Relacionamento](relationships.md) é
que o grupo de campos não é uma tabela independente — ele só existe dentro do
registro que o contém, não pode ser consultado nem relacionado separadamente,
e não aparece como uma tabela própria na lista de tabelas.

Se os dados do grupo precisarem, no futuro, virar uma entidade própria
(consultável, relacionável com outras tabelas), isso exige uma conversão
estrutural — vale pensar com calma, ao modelar, se aquela informação vai
continuar sendo só um detalhe do registro pai ou se vai crescer para algo
maior.

## Campos dentro do grupo

Um grupo de campos pode conter os mesmos [tipos de campo](field-types.md)
disponíveis para uma tabela normal, incluindo campos nativos próprios (como
identificador, criador e datas de criação/atualização daquele item do
grupo).

## Exibição na listagem da tabela pai

Um campo dentro de um grupo pode, opcionalmente, aparecer como coluna na
listagem geral da tabela pai — útil para mostrar um resumo (como "3 itens")
ou um valor agregado sem precisar abrir o registro inteiro.

## Próximos passos

- [Tipos de campo](field-types.md)
- [Relacionamentos](relationships.md)
- [Kanban](../views/kanban.md) — onde grupos de campos aparecem como listas
  de tarefas dentro de um cartão.
