# Segurança

Boas práticas resumidas para quem administra uma instalação própria do
LowCodeJS.

## Credenciais e chaves

- Gere as chaves de segurança da instalação (criptografia de sessão, de
  dados sensíveis) com o gerador próprio incluído no projeto — nunca
  reaproveite chaves de exemplo em produção.
- Guarde a chave de criptografia dos dados sensíveis (Setting e Cofre de
  Senhas) com cuidado: perdê-la torna ilegível tudo o que já foi cifrado com
  ela.

## Permissões

- Conceda a capacidade de **Gerenciar configurações** (acesso à tela de
  Configurações) e o grupo **MASTER** apenas a quem realmente precisa
  administrar a instalação inteira.
- Revise periodicamente os [grupos e capacidades](../settings/users-and-groups.md)
  — principalmente grupos que "englobam" outros, já que a herança de
  permissões pode conceder mais acesso do que o esperado se não for revisada.
- Use [permissões por tabela](../tables/permissions.md) para restringir o
  acesso a dados sensíveis, em vez de depender só das capacidades gerais.
- Para tabelas públicas (acessíveis sem login), confira com atenção quais
  ações estão realmente abertas — visualização, criação, ou ambas.

## Rede

- Sempre que a instalação ficar atrás de um domínio público, utilize HTTPS.
- Restrinja `ALLOWED_ORIGINS` aos domínios que de fato precisam acessar a
  API.
- Ao operar atrás de um proxy reverso, configure-o como confiável (ver
  [Deploy](deployment.md)) para que limites de uso e registros de IP
  funcionem corretamente.

## Armazenamento e backup

- Mantenha backups regulares do banco de dados e do armazenamento de
  arquivos (local ou S3) — ver [Armazenamento](../settings/storage.md).
- Ao usar S3, restrinja as credenciais ao bucket específico da instalação.

## Scripts de automação

- [Scripts](../automations/scripts.md) rodam em um ambiente isolado, com
  acesso restrito a um conjunto controlado de ações — mesmo assim, revise
  scripts escritos por usuários com menos confiança antes de aprová-los em
  tabelas sensíveis, já que eles podem enviar e-mails e criar notificações
  em nome da automação.

## Próximos passos

- [Deploy (self-host)](deployment.md)
- [Permissões de tabela](../tables/permissions.md)
- [Usuários e Grupos](../settings/users-and-groups.md)
