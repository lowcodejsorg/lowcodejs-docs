# Ambientes (Workspaces)

Um **ambiente** (workspace) é um espaço de trabalho isolado dentro da mesma
instalação: tem seu próprio conjunto de tabelas, usuários-membro e menus,
completamente separado dos demais ambientes.

## Para que servem

- Separar **clientes ou departamentos diferentes** na mesma instalação, sem
  que um veja os dados do outro.
- Manter um ambiente de **testes** isolado do de produção.
- Organizar projetos ou times que não precisam compartilhar nenhuma tabela
  entre si.

## O ambiente padrão

Toda instalação tem um **ambiente padrão**, criado automaticamente — é para
onde o usuário é direcionado ao entrar, caso não tenha vínculo com nenhum
outro ambiente específico.

## Criando e administrando ambientes

Em **Ambientes**, quem tem a capacidade de gerenciar ambientes pode:

- Criar um ambiente novo, vazio ou **clonado** a partir da estrutura de outro
  ambiente existente (tabelas, campos, relacionamentos e menus — nunca os
  registros).
- Listar os ambientes existentes e trocar entre eles.
- Gerenciar quem tem acesso a cada ambiente.

## Trocando de ambiente

Um usuário pode ter acesso a vários ambientes diferentes e alternar entre
eles pela interface, sem precisar fazer login novamente — o conteúdo exibido
(tabelas, menus, dados) muda conforme o ambiente ativo.

## Próximos passos

- [Usuários e Grupos](users-and-groups.md)
- [O que é uma tabela](../tables/overview.md)
