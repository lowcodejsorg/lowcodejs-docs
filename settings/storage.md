# Armazenamento

Controla onde os arquivos enviados pelos usuários (anexos de campos do tipo
Arquivo, logos, imagens) ficam guardados.

## Opções disponíveis

| Opção | Onde ficam os arquivos |
| --- | --- |
| **Local** | No próprio servidor da instalação |
| **S3** | Em um serviço de armazenamento compatível com S3 (AWS S3 ou qualquer serviço compatível), configurado com endereço, região, bucket e credenciais próprias |

Por padrão, uma instalação nova usa armazenamento local. Ativar o S3 exige
informar o endpoint, a região, o nome do bucket e as credenciais de acesso
daquele serviço.

## Upload de arquivos grandes

O envio de arquivos é feito em partes, com retomada automática caso a
conexão seja interrompida no meio do envio — não é preciso recomeçar do zero
um upload grande que falhou perto do fim.

## Trocando de driver com arquivos já existentes

Quando o administrador alterna entre armazenamento local e S3 (ou
vice-versa) numa instalação que já tem arquivos enviados, os arquivos
existentes continuam acessíveis — a plataforma localiza arquivos antigos no
driver anterior enquanto a migração para o novo não é feita.

## Quem configura

Apenas o usuário **MASTER**, pela tela de Configurações.

## Próximos passos

- [Tela de Configurações](overview.md)
- [Tipos de campo](../tables/field-types.md) — veja o campo Arquivo.
