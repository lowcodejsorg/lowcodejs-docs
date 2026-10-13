# Deploy (self-host)

Resumo de como colocar uma instalação do LowCodeJS no ar, para quem
administra o servidor. Para o passo a passo de desenvolvimento local, veja
[Instalação](../getting-started/installation.md).

## Imagens oficiais

A cada atualização da plataforma, as imagens oficiais são publicadas
automaticamente e podem ser usadas diretamente, sem precisar compilar nada:

```bash
docker compose -f docker-compose.oficial.yml up -d
```

Esse compose usa apenas as imagens prontas (`:latest`), ideal para quem só
quer hospedar a plataforma sem mexer no código-fonte.

## Variáveis de ambiente essenciais em produção

| Variável | Para que serve |
| --- | --- |
| `APP_SERVER_URL` | URL pública da API |
| `APP_CLIENT_URL` | URL pública do site |
| `DATABASE_URL` | Endereço do banco de dados interno |
| `DATA_POSTGRES_URL` | Endereço do PostgreSQL das tabelas de dados (opcional, decidido uma única vez na instalação) |
| `ALLOWED_ORIGINS` | Domínios autorizados a acessar a API |

A maior parte da configuração de produto (branding, idioma, armazenamento,
e-mail, IA) **não** é feita por variável de ambiente — é configurada pela
interface depois que a aplicação já está no ar, em **Configurações**.

## Atrás de um proxy reverso

Quando a instalação roda atrás de um proxy reverso ou balanceador (Traefik,
Nginx, um serviço de deploy gerenciado), é importante que o proxy esteja
configurado como confiável para que a aplicação reconheça corretamente o IP
de origem das requisições — relevante tanto para limites de uso quanto para
qualquer automação que registre o IP do usuário.

## Atualizações

```bash
docker compose pull
docker compose up -d
```

As migrações de estrutura necessárias são aplicadas automaticamente ao
subir uma versão nova — não é preciso rodar nenhum comando manual de
migração em uma atualização normal.

## Backup

Uma rotina de backup confiável cobre:

- O volume do banco de dados.
- O diretório de armazenamento local, ou o bucket S3 configurado (ver
  [Armazenamento](../settings/storage.md)).

## Próximos passos

- [Instalação](../getting-started/installation.md)
- [Segurança](security.md)
