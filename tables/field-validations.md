# Validações de campo

Além do [tipo](field-types.md) e do [formato](field-formats.md), um campo
pode receber uma ou mais **regras de validação**, que impedem salvar um
registro com um valor inválido. As regras se somam: um campo pode ter
"Obrigatório" e "Único" ao mesmo tempo, por exemplo.

Algumas regras são conferidas instantaneamente (no navegador, enquanto você
digita); outras precisam checar o banco de dados e só são conferidas ao
salvar — isso é só um detalhe técnico de quando o aviso aparece, não muda
como a regra é configurada.

## Regras disponíveis

| Validação | O que exige |
| --- | --- |
| **Não vazio** | O campo não pode ficar em branco |
| **É e-mail** | O valor precisa ser um e-mail válido |
| **É numérico** | Apenas números |
| **É alfanumérico** | Apenas letras e números |
| **Está no intervalo** | O valor (numérico) precisa estar entre um mínimo e um máximo configurados |
| **É IBAN** | Formato de conta bancária internacional (IBAN) válido |
| **É diferente de** | O valor não pode ser igual a um ou mais valores informados na configuração da regra |
| **É URL** | O valor precisa ser um endereço web válido |
| **É telefone** | O valor precisa ter formato de telefone válido |
| **É CPF** | O valor precisa ser um CPF válido |
| **É CNPJ** | O valor precisa ser um CNPJ válido |
| **É único** | Não pode haver outro registro da mesma tabela com o mesmo valor nesse campo |
| **Valores únicos** | Em um campo de múltiplos valores, não permite repetir o mesmo valor dentro da própria lista |
| **E-mail existe** | O e-mail informado precisa pertencer a um usuário já cadastrado na instalação |
| **Usuário existe** | O valor precisa corresponder a um usuário já cadastrado |
| **Grupo existe** | O valor precisa corresponder a um grupo de permissão já cadastrado |

## Exemplos práticos

- Um campo **E-mail do contato** (texto curto, formato E-mail) ganhando as
  validações **Não vazio** + **É único** garante que todo registro tenha um
  e-mail preenchido e que não haja dois contatos com o mesmo e-mail.
- Um campo **Desconto (%)** ganhando **Está no intervalo** (0 a 100) impede
  valores fora da faixa permitida.
- Um campo **Status anterior** ganhando **É diferente de** (configurado com
  o valor "Cancelado") impede que alguém marque esse status de volta.
- Um campo **Aprovador** (tipo Usuário) ganhando **Usuário existe** garante
  que só seja possível apontar para uma conta de fato cadastrada.

## Próximos passos

- [Tipos de campo](field-types.md)
- [Formatos de campo](field-formats.md)
- [Permissões de tabela](permissions.md)
