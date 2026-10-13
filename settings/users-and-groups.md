# Usuários e Grupos

A plataforma controla o acesso por **grupos de usuário** configuráveis, com
capacidades gerais e hierarquia entre si — em vez de um punhado fixo de
papéis.

## Usuários

Em **Usuários**, o administrador cadastra e gerencia as contas da
instalação: nome, e-mail, senha e os grupos aos quais cada conta pertence.
Um usuário pode pertencer a **vários grupos** ao mesmo tempo, acumulando as
permissões de todos eles.

## Grupos

Em **Grupos**, você cria e configura os grupos de permissão. Cada grupo
define:

- Um conjunto de **capacidades de área** (ver abaixo).
- Quais outros grupos ele **engloba** — um grupo que engloba outro herda
  automaticamente todas as permissões do grupo englobado, formando uma
  hierarquia.

### Grupos padrão do sistema

A instalação já vem com quatro grupos padrão, mantidos por compatibilidade e
usados como base de hierarquia:

| Grupo | Nível |
| --- | --- |
| **MASTER** | Acesso total, incluindo Configurações da instalação |
| **ADMINISTRATOR** | Acesso administrativo amplo |
| **MANAGER** | Acesso de gestão intermediário |
| **REGISTERED** | Usuário comum registrado |

Esses quatro continuam existindo, mas o controle efetivo de acesso hoje é
feito por grupos customizados com capacidades e permissões específicas — os
quatro grupos padrão podem ser usados como ponto de partida ou "englobados"
por um grupo customizado.

## Capacidades de área

Uma **capacidade** libera o acesso a uma área inteira do sistema. Capacidades
podem ser atribuídas a qualquer grupo, customizado ou padrão:

| Capacidade | Libera |
| --- | --- |
| Gerenciar usuários | Cadastro e gestão de contas |
| Gerenciar menu | Configuração dos itens de menu |
| Gerenciar grupos de usuário | Criação e edição de grupos |
| Gerenciar configurações | Acesso à tela de Configurações |
| Gerenciar ferramentas | Ativar/desativar ferramentas |
| Gerenciar plugins | Configurar plugins instalados |
| Gerenciar chat | Administração do Assistente de IA/chat |
| Gerenciar ambientes | Criar e administrar workspaces |
| Uma capacidade por ferramenta | Usar cada ferramenta específica (Clonar Tabela, Importar/Exportar, Gerador de Dados de Teste, Importar de Outra Instalação, BPM Modeler, Cofre de Senhas) |

## Permissões específicas de cada tabela

Além das capacidades gerais, cada tabela tem suas próprias permissões (quem
vê, cria, edita e remove tabela/campo/registro) e pode ter membros com
perfis específicos — isso é configurado dentro de cada tabela, não aqui. Veja
[Permissões de tabela](../tables/permissions.md) para o detalhamento
completo.

## Como as duas camadas se combinam

Para fazer algo numa tabela específica, o usuário geralmente precisa de
**duas coisas ao mesmo tempo**: a capacidade geral correspondente (vinda do
fecho de grupos que ele pertence) e a permissão específica daquela tabela
liberando a ação para o seu grupo. Usuários MASTER, donos de tabela e
membros com perfil específico não passam por essa checagem dupla.

## Próximos passos

- [Permissões de tabela](../tables/permissions.md)
- [Ambientes (Workspaces)](workspaces.md)
