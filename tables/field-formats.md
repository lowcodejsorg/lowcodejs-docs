# Formatos de campo

Alguns [tipos de campo](field-types.md) aceitam um **formato**, que refina
como o valor é interpretado, validado e exibido. O formato é escolhido na
configuração do campo, logo após o tipo.

## Formatos de Texto curto

| Formato | O que valida/formata | Exemplo |
| --- | --- | --- |
| **Alfanumérico** | Letras e números, sem regra adicional | `Produto123` |
| **Número inteiro** | Apenas números inteiros | `42` |
| **Decimal** | Números com casas decimais | `19,90` |
| **URL** | Endereço web válido | `https://exemplo.com` |
| **E-mail** | Endereço de e-mail válido | `contato@exemplo.com` |
| **Senha** | Campo de senha (mascarado na digitação) | `••••••••` |
| **Telefone** | Número de telefone, com máscara | `(62) 99999-0000` |
| **CNPJ** | Documento de empresa brasileiro, com máscara e validação | `00.000.000/0001-00` |
| **CPF** | Documento de pessoa física brasileiro, com máscara e validação | `000.000.000-00` |

## Formatos de Texto longo

| Formato | O que é |
| --- | --- |
| **Texto rico** | Editor completo, com formatação (negrito, itálico, listas, links, menções) |
| **Texto simples** | Apenas texto puro, sem formatação |

## Formatos de Data

O campo Data aceita vários formatos de exibição, combinando a ordem de
dia/mês/ano, a presença de horário e o separador (barra ou hífen):

| Formato | Exemplo |
| --- | --- |
| `dd/MM/yyyy` | 31/12/2026 |
| `MM/dd/yyyy` | 12/31/2026 |
| `yyyy/MM/dd` | 2026/12/31 |
| `dd/MM/yyyy HH:mm:ss` | 31/12/2026 23:59:59 |
| `MM/dd/yyyy HH:mm:ss` | 12/31/2026 23:59:59 |
| `yyyy/MM/dd HH:mm:ss` | 2026/12/31 23:59:59 |
| `dd-MM-yyyy` | 31-12-2026 |
| `MM-dd-yyyy` | 12-31-2026 |
| `yyyy-MM-dd` | 2026-12-31 |
| `dd-MM-yyyy HH:mm:ss` | 31-12-2026 23:59:59 |
| `MM-dd-yyyy HH:mm:ss` | 12-31-2026 23:59:59 |
| `yyyy-MM-dd HH:mm:ss` | 2026-12-31 23:59:59 |

O formato escolhido afeta apenas a exibição e a máscara de digitação — o
valor por trás continua sendo uma data normal, que pode ser usada em
filtros, ordenação e nas visualizações de [Calendário e Gantt](../views/calendar-gantt.md).

## Próximos passos

- [Tipos de campo](field-types.md)
- [Validações de campo](field-validations.md)
