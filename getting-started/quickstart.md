# Primeiros Passos

Um roteiro rápido para criar sua primeira tabela e começar a usar a
plataforma.

## 1. Acesse a plataforma

Se você já tem uma instalação rodando, acesse a URL fornecida e entre com
suas credenciais. Se ainda não existe nenhuma instalação, a primeira pessoa a
acessar passa pelo **Setup Wizard**, que pede:

- Nome e descrição da instalação (branding).
- Conta do usuário **MASTER** (o primeiro administrador).
- Qual banco guardará as tabelas de dados (ver
  [Instalação](installation.md)) — decisão única e definitiva.
- Armazenamento de arquivos: local ou S3.
- Servidor de e-mail (opcional nesse momento, pode ser configurado depois).

## 2. Crie sua primeira tabela

No menu lateral, abra **Tabelas** e use o botão de criar nova tabela. Você
pode:

- Começar do zero, dando um nome (o identificador/slug é gerado
  automaticamente a partir dele).
- Usar um **modelo pronto** (Kanban, Cartões, Mosaico, Documento, Fórum ou
  Calendário), direto na tela de criação — já vem com os campos certos para
  aquele estilo de visualização.

## 3. Adicione campos

Dentro da tabela recém-criada, abra a gestão de campos e adicione os campos
que fazem sentido para o seu caso — nome do cliente, status, data de entrega,
responsável, etc. Veja a lista completa em
[Tipos de campo](../tables/field-types.md).

Para cada campo você pode configurar:

- Se é obrigatório.
- Se aceita múltiplos valores.
- **Formato** (ver [Formatos de campo](../tables/field-formats.md)), quando
  aplicável — por exemplo, um campo de texto curto pode ser um e-mail, CPF ou
  URL.
- **Validações** (ver [Validações de campo](../tables/field-validations.md))
  — regras extras como "não vazio" ou "só números".
- Em quais telas o campo aparece (lista, formulário, detalhe) — ver
  [Permissões de tabela](../tables/permissions.md).

## 4. Crie o primeiro registro

Use o botão de criar registro. O formulário é montado automaticamente a
partir dos campos configurados, na ordem definida.

## 5. Escolha como visualizar

Toda tabela nasce em modo **Lista**, mas pode trocar de visualização a
qualquer momento — veja a [Visão geral das 9 visualizações](../views/overview.md)
para escolher a que melhor se encaixa no seu caso (Kanban para fluxo de
tarefas, Calendário para eventos, Fórum para discussões, etc.).

## 6. Defina quem pode ver e editar

Configure as [permissões da tabela](../tables/permissions.md): quem pode ver,
criar, editar e remover tabela, campos e registros, além de convidar
**membros** com perfis específicos (Dono, Admin, Editor, Colaborador,
Visualizador).

## 7. Automatize (opcional)

Se a tabela precisa de alguma lógica — enviar um e-mail quando um registro é
criado, calcular um valor antes de salvar — veja
[Scripts](../automations/scripts.md). Para processos mais elaborados, com
várias etapas e tarefas, veja o
[Modelador de Processos (BPM)](../automations/bpm-modeler.md).

## Próximos passos

- [O que é uma tabela](../tables/overview.md) — aprofunde no conceito.
- [Tipos de campo](../tables/field-types.md) — veja a lista completa.
- [Visão geral das 9 visualizações](../views/overview.md).
