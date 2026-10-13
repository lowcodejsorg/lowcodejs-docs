# Scripts (beforeSave / afterSave / onLoad)

Tabelas podem ter **scripts** próprios — pequenos trechos de código que
rodam automaticamente em determinados momentos do ciclo de vida de um
registro, sem precisar de um desenvolvedor no time para configurar a maior
parte dos casos. Os scripts rodam em um ambiente isolado e seguro no
servidor, com um conjunto limitado e controlado de ações disponíveis.

## Quando um script roda

| Momento | Quando dispara |
| --- | --- |
| **beforeSave** | Antes de um registro ser salvo — bom para validar, calcular ou ajustar valores antes de gravar |
| **afterSave** | Depois que um registro foi salvo com sucesso — bom para enviar e-mails, criar notificações ou disparar efeitos colaterais |
| **onLoad** | Ao carregar o formulário de um registro |

O script também sabe identificar o contexto da ação: se é um registro novo,
uma edição, uma exclusão ou apenas o carregamento do formulário.

## O que um script pode fazer

Dentro do ambiente seguro de execução, o script tem acesso a um conjunto
específico de recursos:

| Recurso | Para que serve |
| --- | --- |
| **Campos do registro** | Ler e escrever valores dos campos do registro atual |
| **Contexto da ação** | Saber se é criação, edição, exclusão ou carregamento, quem é o usuário, qual tabela, entre outros dados do momento |
| **E-mail** | Enviar e-mails (diretamente ou a partir de um modelo) |
| **Usuários** | Resolver ids de campos de usuário em nome/e-mail |
| **Notificações** | Criar notificações internas, que aparecem em tempo real para quem for destinatário |
| **Utilitários de data e hash** | Data/hora atual, formatação de data, geração de identificadores e códigos |
| **Console** | Registrar mensagens de depuração durante a execução do script |

Por segurança, um script **não** tem acesso à rede externa, ao sistema de
arquivos ou a dados de outras tabelas além do registro atual — apenas às
APIs listadas acima.

## Exemplos de uso

- **beforeSave**: calcular o valor total de um pedido a partir dos itens,
  ou impedir que um registro seja salvo com uma combinação de valores
  inválida que as [validações de campo](../tables/field-validations.md)
  sozinhas não cobrem.
- **afterSave**: enviar um e-mail de confirmação ao criar um pedido, ou
  notificar o responsável quando o status de uma tarefa muda.
- **onLoad**: pré-preencher um campo com um valor calculado no momento em
  que o formulário é aberto.

## Próximos passos

- [Modelador de Processos (BPM)](bpm-modeler.md) — para automações mais
  elaboradas, com várias etapas conectadas visualmente, que geram esses
  scripts automaticamente.
