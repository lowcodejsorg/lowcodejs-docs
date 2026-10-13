# Instalação

Guia para hospedar sua própria instância do LowCodeJS (self-host) via Docker
Compose. Esta seção assume familiaridade básica com linha de comando e
Docker — é voltada a quem vai administrar a instalação, não ao uso do dia a
dia da plataforma.

## Pré-requisitos

- **Docker** e **Docker Compose**.
- **Git Bash** no Windows (para rodar os scripts `.sh` de configuração).

## Configuração inicial

Na raiz do projeto:

```bash
chmod +x ./setup.sh
./setup.sh
```

O script cria os arquivos de ambiente (`.env`) e gera automaticamente as
chaves de segurança necessárias (criptografia de sessão e de dados
sensíveis). Rodar de novo não sobrescreve um `.env` já existente, a menos que
se use `./setup.sh --force`.

**Nunca** reaproveite chaves de exemplo em produção — gere as suas com o
script.

## Subindo os serviços

```bash
docker compose up -d
```

Isso sobe os serviços essenciais: banco de dados, cache, backend (API) e
frontend. Depois, rode a carga inicial de dados (grupos e permissões
padrão):

```bash
docker exec -it low-code-js-api pnpm seed
```

| Serviço | URL padrão |
| --- | --- |
| Aplicação (frontend) | http://localhost:5173 |
| API (backend) | http://localhost:3000 |

Na primeira visita, o **Setup Wizard** guia a configuração inicial: nome da
instalação, conta MASTER, banco das tabelas de dados, armazenamento e
e-mail — tudo pela interface, sem precisar editar arquivo nenhum depois
disso.

## Serviços opcionais

A plataforma sobe com um núcleo mínimo; alguns recursos são opcionais e
ativados por "profile" do Docker Compose.

### Assistente de IA

```bash
docker compose --profile ai up -d
```

Depois de subir, configure a chave de API e ative o assistente em
**Configurações** → [Assistente de IA](../settings/ai-assistant.md) (usuário
MASTER).

### Banco das tabelas de dados: Mongo ou Postgres

O sistema (usuários, estrutura das tabelas, configurações) sempre vive no
mesmo banco interno. Mas os **dados que você cadastra** nas tabelas podem
viver nesse mesmo banco ou em um PostgreSQL separado — essa escolha é feita
uma única vez, no Setup Wizard de uma instalação nova, e não pode ser
alterada depois.

Para habilitar a opção de usar PostgreSQL, suba o serviço correspondente
antes do primeiro acesso:

```bash
docker compose --profile data-postgres up -d
```

Com o serviço disponível, a etapa "Banco de dados" do Setup Wizard passa a
oferecer a escolha entre os dois. Sem ele, a instalação usa só o banco
interno.

### Armazenamento em nuvem (S3)

Não exige nenhum serviço adicional no Docker Compose — é configurado
inteiramente pela interface, em **Configurações** →
[Armazenamento](../settings/storage.md). Sem ativar, os arquivos enviados
ficam guardados localmente, no próprio volume da instalação.

## Atualizando a instalação

As imagens oficiais (`:latest`) são publicadas automaticamente a cada
atualização da plataforma. Para atualizar uma instalação existente:

```bash
docker compose pull
docker compose up -d
```

## Variáveis de ambiente essenciais

A maior parte da configuração (nome da instalação, logos, idioma,
armazenamento, e-mail, IA) é feita pela interface depois da instalação — não
por variável de ambiente. O `.env` cobre apenas o que é necessário **antes**
da aplicação subir:

| Variável | Descrição |
| --- | --- |
| `APP_SERVER_URL` | URL pública de onde a API responde |
| `APP_CLIENT_URL` | URL pública de onde o site responde |
| `DATABASE_URL` | Endereço do banco de dados interno |
| `DATA_POSTGRES_URL` | Endereço do PostgreSQL das tabelas de dados (opcional) |

## Backup e manutenção

Como toda a estrutura e os dados das tabelas ficam no banco de dados
(interno e, opcionalmente, PostgreSQL) e os arquivos enviados ficam no
armazenamento configurado (local ou S3), uma rotina de backup confiável
cobre:

- O volume do banco de dados.
- O diretório de armazenamento local (quando não estiver usando S3) — ou o
  próprio bucket S3, se for o caso.

## Problemas comuns

**Erro de permissão ao rodar `setup.sh`**:

```bash
chmod +x ./setup.sh ./credential-generator.sh
```

**Serviço não sobe**: confira com `docker ps` se os containers estão de pé e
`docker compose logs` para detalhes do erro.

Veja também [Deploy (self-host)](../admin/deployment.md) para variáveis
adicionais de produção e [Segurança](../admin/security.md) para boas
práticas de administração.
