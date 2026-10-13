# Visão geral das ferramentas

**Ferramentas** são utilitários que aparecem no menu **Ferramentas**, cada um
resolvendo uma tarefa específica de administração de tabelas ou dados. Assim
como [plugins e módulos](#plugins-e-modulos-o-resto-das-extensoes), são um
tipo de **extensão** — podem ser ativadas ou desativadas pelo usuário MASTER
em **Extensões**, e algumas exigem uma capacidade própria liberada no grupo
do usuário para aparecer.

## Ferramentas disponíveis

| Ferramenta | O que faz |
| --- | --- |
| [Clonar Tabela](clone-table.md) | Cria tabelas novas copiando a estrutura de tabelas existentes, com ou sem os dados — inclui os modelos prontos (Kanban, Cartões, Mosaico, Documento, Fórum, Calendário) |
| [Importar / Exportar Tabela](import-export.md) | Exporta uma tabela para um arquivo, ou importa uma tabela a partir de um arquivo exportado |
| [Gerador de Dados de Teste](test-data-generator.md) | Gera registros fictícios em massa numa tabela, para testar ou demonstrar |
| [Cofre de Senhas](password-vault.md) | Guarda segredos (senhas, chaves) de forma criptografada, organizados em canais privados |
| [Importar de Outra Instalação](installation-import.md) | Conecta numa instalação LowCodeJS existente e traz tabelas e dados dela |
| Modelador de Processos (BPM) | Ver [Automações → BPM Modeler](../automations/bpm-modeler.md) |
| Transcrição de Documentos | Envia documentos (como CNH, comprovante de endereço) para extração automática de dados estruturados |

## Plugins e módulos (o resto das extensões)

Além das ferramentas, a plataforma também suporta **plugins** (pequenos
botões ou widgets inseridos em pontos específicos da interface, como
"Copiar link do registro" ou "Imprimir tabela") e **módulos** (telas maiores
com URL própria, como o Dashboard ou o Repositório de Apps). Tudo isso é
gerenciado na mesma tela de **Extensões**.

| Tipo | O que é | Exemplo |
| --- | --- | --- |
| **Ferramenta** | Utilitário listado no menu Ferramentas | Clonar Tabela |
| **Plugin** | Botão/ação inserida num ponto específico da interface | Copiar link do registro, Imprimir tabela |
| **Módulo** | Tela própria, com URL dedicada | Dashboard, Repositório de Apps |

## Ativando uma ferramenta

Em **Extensões** (restrito ao usuário MASTER), cada extensão disponível
aparece com um botão de ativar/desativar. Uma vez ativada, ela passa a
aparecer para os usuários que tiverem a capacidade correspondente liberada no
seu grupo — ver [Usuários e Grupos](../settings/users-and-groups.md).

## Próximos passos

- [Clonar Tabela](clone-table.md)
- [Importar / Exportar Tabela](import-export.md)
