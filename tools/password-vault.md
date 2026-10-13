# Cofre de Senhas

Um cofre para guardar senhas e outros segredos (chaves de API, credenciais de
acesso) de forma criptografada, organizado em canais privados — pensado para
compartilhar credenciais dentro de uma equipe sem expor o valor em texto
simples.

## Canais privados

Os segredos são organizados em canais, que por padrão são privados — visíveis
apenas para quem foi explicitamente adicionado àquele canal.

## Segurança dos dados

Os segredos ficam **criptografados em repouso** no banco de dados: mesmo
quem tiver acesso direto ao banco não consegue ler o valor sem a chave de
criptografia da instalação, configurada pelo administrador do servidor.

## Quem pode usar

O acesso ao Cofre de Senhas depende de uma capacidade própria, liberada pelo
usuário MASTER no grupo do usuário — por padrão, não vem habilitado para
todo mundo.

## Quando usar

- Compartilhar a senha de um serviço externo entre os membros de uma equipe.
- Guardar chaves de API ou tokens que várias pessoas precisam consultar, sem
  depender de planilhas ou mensagens.

## Próximos passos

- [Usuários e Grupos](../settings/users-and-groups.md) — como liberar a
  capacidade de uso de uma ferramenta para um grupo.
