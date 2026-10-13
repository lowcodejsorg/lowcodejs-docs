# Visão Geral da Interface

Um tour rápido pelas áreas principais da plataforma, vistas de quem está
logado.

## Menu lateral

O menu lateral é o ponto de partida para quase tudo. Os itens disponíveis
dependem do grupo e das capacidades do usuário logado (ver
[Permissões de tabela](../tables/permissions.md) e
[Usuários e Grupos](../settings/users-and-groups.md)) — quem não é MASTER ou
ADMINISTRATOR, por exemplo, pode não ver "Usuários" ou "Configurações".

| Item do menu | Para que serve |
| --- | --- |
| **Dashboard** | Painel com gráficos e cartões de métrica sobre os dados do ambiente |
| **Tabelas** | Lista de todas as tabelas que você tem acesso; ponto de entrada para criar, abrir e configurar tabelas |
| **Usuários** | Cadastro de contas de usuário da instalação |
| **Grupos** | Grupos de permissão (RBAC) — quem pode fazer o quê |
| **Menus** | Organização da navegação: itens de menu customizados, inclusive apontando para módulos de extensão ou páginas |
| **Ferramentas** | Utilitários como Clonar Tabela, Importar/Exportar, Gerador de Dados de Teste, BPM Modeler |
| **Extensões** | Tela de gestão (ativar/desativar) dos plugins, módulos e ferramentas instalados — restrita ao MASTER |
| **Repositório** | Catálogo de apps prontos para instalar com um clique |
| **Ambientes** | Lista dos workspaces da instalação e troca entre eles |
| **Configurações** | Configurações gerais da instalação — ver [Tela de Configurações](../settings/overview.md) |
| **Histórico** | Auditoria: quem fez o quê e quando |
| **Perfil** | Dados da própria conta |

Itens de menu customizados (criados em **Menus**) também podem apontar para
tabelas específicas, páginas ou módulos de uma extensão — o que aparece no
menu lateral do dia a dia de um app pronto normalmente é um subconjunto
pensado por quem montou a aplicação.

## Áreas de uma tabela

Ao abrir uma tabela, você normalmente encontra:

- A **visualização atual** (lista, kanban, calendário...) com os registros.
- Um botão para **trocar de visualização** ou configurar uma nova.
- **Filtros** na lateral ou no topo, para refinar o que aparece.
- Um botão para **criar um novo registro**, que abre o formulário montado a
  partir dos campos configurados na tabela.
- Um menu de configuração da tabela — campos, permissões, membros, métodos
  (automações) — normalmente restrito a quem administra aquela tabela.

## Notificações

Um sino no cabeçalho mostra notificações em tempo real: menções em
comentários (Kanban, Fórum), atribuições de membro e outros eventos
relevantes configurados por automação.

## Assistente de IA

Quando ativado pelo MASTER em Configurações, um chat fica disponível para
conversar sobre os dados da instalação, enviar imagens e documentos (PDF) e
disparar ações através de ferramentas da própria plataforma.

## Próximos passos

- [Primeiros Passos](quickstart.md) — crie sua primeira tabela.
- [O que é uma tabela](../tables/overview.md) — o conceito central da
  plataforma.
