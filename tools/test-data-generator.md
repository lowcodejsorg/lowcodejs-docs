# Gerador de Dados de Teste

Gera registros fictícios em massa para uma tabela selecionada — útil para
testar uma visualização, apresentar um protótipo ou simplesmente povoar uma
tabela nova antes de usá-la de verdade.

## Como funciona

Escolha a tabela de destino e a quantidade de registros desejada. A
ferramenta gera valores plausíveis para cada campo, de acordo com o seu tipo
e formato — por exemplo, um campo de e-mail recebe um e-mail de formato
válido, um campo de CPF recebe um CPF com formato correto, um campo de data
recebe uma data dentro de uma faixa razoável.

## Limite de geração

A geração tem um teto de tamanho estimado (volume de dados a inserir). Se a
quantidade solicitada ultrapassar esse teto, a ferramenta ajusta o
comportamento para não sobrecarregar a instalação.

## Quando usar

- Testar como uma [visualização](../views/overview.md) se comporta com
  muitos registros.
- Demonstrar uma tabela recém-criada antes de usá-la com dados reais.
- Popular um ambiente de testes rapidamente.

Os registros gerados são dados normais da tabela — podem ser filtrados,
editados ou removidos (inclusive em lote) como qualquer outro registro depois
de cumprirem seu papel de teste.

## Próximos passos

- [Clonar Tabela](clone-table.md)
- [Visão geral das 9 visualizações](../views/overview.md)
