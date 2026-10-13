# Importar de Outra Instalação

Conecta a uma instalação LowCodeJS de origem (em modo somente leitura),
permite ver as tabelas e campos dela e pareá-los com tabelas do ambiente de
destino, para trazer estrutura e dados de uma instalação para outra.

## Quando usar

- **Migrar** de uma instalação antiga para uma nova, sem recriar tudo
  manualmente.
- **Consolidar** dados de instalações separadas em uma só.
- Trazer uma ou mais tabelas específicas de outra instalação, sem precisar
  exportar/importar arquivo por arquivo.

## Como funciona, em linhas gerais

1. Informe os dados de conexão da instalação de origem.
2. A ferramenta lê, em modo somente leitura, as tabelas e campos disponíveis
   lá.
3. Você pareia cada tabela (e campo) de origem com uma tabela de destino no
   ambiente atual — criando uma nova, se for o caso.
4. A importação traz a estrutura e os dados conforme o pareamento definido.

## Diferença para Importar / Exportar Tabela

[Importar / Exportar Tabela](import-export.md) trabalha com um **arquivo**
isolado, exportado manualmente de uma tabela por vez. Esta ferramenta, em
vez disso, se conecta diretamente a outra instalação **inteira**, permitindo
trazer várias tabelas de uma vez com o pareamento feito pela interface.

## Próximos passos

- [Importar / Exportar Tabela](import-export.md)
- [Clonar Tabela](clone-table.md)
