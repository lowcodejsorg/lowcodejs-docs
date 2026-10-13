# Importar / Exportar Tabela

Exporta uma tabela para um arquivo (estrutura, dados, ou ambos), que depois
pode ser importado em outro ambiente ou em outra instalação da plataforma.

## Exportando

Escolha a tabela e o que incluir no arquivo:

- Apenas a **estrutura** (campos, configurações, permissões) — útil para
  levar um modelo de tabela para outro ambiente sem levar os dados.
- **Estrutura e dados** — uma cópia completa da tabela, pronta para ser
  restaurada em outro lugar.

O resultado é um arquivo que pode ser guardado como backup pontual ou
compartilhado com outra instalação.

## Importando

Selecione um arquivo exportado anteriormente pela própria plataforma e a
ferramenta recria a tabela (e os dados, se estiverem incluídos) no ambiente
atual.

## Quando usar

- Levar uma tabela de um ambiente de testes para produção (ou vice-versa).
- Fazer backup pontual de uma tabela específica antes de uma mudança
  arriscada na sua estrutura.
- Compartilhar um modelo de tabela entre instalações diferentes do
  LowCodeJS.

Para trazer **várias** tabelas de uma instalação LowCodeJS inteira diferente
(não apenas um arquivo isolado), veja
[Importar de Outra Instalação](installation-import.md).

## Próximos passos

- [Clonar Tabela](clone-table.md) — para duplicar uma tabela dentro do mesmo
  ambiente.
- [Importar de Outra Instalação](installation-import.md)
