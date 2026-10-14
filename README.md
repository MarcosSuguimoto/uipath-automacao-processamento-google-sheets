# RPA - Processamento de Solicitações via Google Sheets

Projeto de automação de processos de negócios desenvolvido no **UiPath Studio** para leitura, validação, processamento e auditoria de solicitações registradas em uma planilha do Google Sheets.

---

## 📌 Objetivos do Projeto

- **Automação End-to-End**: Leitura automatizada de solicitações diretamente do Google Sheets.
- **Validação de Dados**: Filtragem rigorosa de e-mails inválidos (ausência do caractere `@`) e valores de solicitação zerados ou negativos ($\le 0$).
- **Rastreabilidade e Auditoria**: Registro detalhado dos motivos de falha em uma aba dedicada (`Erros`) para facilitar a correção pelos usuários.
- **Resiliência**: Tratamento de falhas temporárias no processamento através de mecanismo de re-tentativa (`Retry Scope`).
- **Arquitetura Modular**: Separação clara de responsabilidades em fluxos de trabalho reutilizáveis e independentes (`.xaml`).

---

## 🏗️ Arquitetura do Projeto

O projeto utiliza uma arquitetura modular orquestrada pelo arquivo `Main.xaml`:

```text
RPA_Processamento_Solicitacoes_GoogleSheets/
├── Main.xaml                      # Orquestrador principal do projeto
├── Fluxo_De_Trabalho/
│   ├── Leitura_Planilha.xaml      # Módulo de integração e leitura do Google Sheets
│   ├── Validacao_Dados.xaml       # Módulo de regras de validação de dados e identificação de erros
│   ├── Processamento_Regras.xaml  # Módulo de execução das regras de negócio com Retry Scope
│   └── Atualizacao_Resultados.xaml# Módulo de gravação das abas Resultado e Erros
└── README.md                      # Documentação técnica do projeto
```

---

## 🔄 Fluxo de Execução

1. **`Leitura_Planilha.xaml`**
   - Conecta ao Google Drive via atividade **Google Workspace Scope**.
   - Lê a aba `"Solicitações"` utilizando a atividade **Read Range**.
   - Retorna a `DataTable` contendo todas as solicitações pendentes (`out_dtSolicitacoes`).

2. **`Validacao_Dados.xaml`**
   - Recebe `in_dtSolicitacoes` e clona a estrutura para criar `out_dtValidos` e `out_dtInvalidos`.
   - Adiciona a coluna `Motivo_Erro` na tabela de inválidos.
   - Percorre cada linha validando as condições de e-mail e valor.
   - Em caso de falha, determina a justificativa exata via expressão inline (`If(...)`) e adiciona a linha à tabela `out_dtInvalidos`.

3. **`Processamento_Regras.xaml`**
   - Recebe `in_dtValidos` e inicializa a estrutura de resultados (`out_dtResultados`).
   - Processa cada solicitação válida dentro de um bloco **Retry Scope** (3 tentativas com intervalo de 2 segundos).
   - Preenche os dados de execução (`ID`, `Cliente`, `Tipo`, `Valor`, Status `"Sucesso"`, Data/Hora atual e Mensagem de confirmação).

4. **`Atualizacao_Resultados.xaml`**
   - Conecta à planilha e grava os registros de `in_dtResultados` na aba `"Resultado"` (modo **Substituir / Overwrite**).
   - Grava os registros de `in_dtInvalidos` juntamente com a coluna `Motivo_Erro` na aba `"Erros"` (modo **Substituir / Overwrite**).

---

## 📊 Estrutura das Abas no Google Sheets

| Aba | Função | Colunas Principais |
| :--- | :--- | :--- |
| **Solicitações** | Entrada de Dados | `ID_Solicitacao`, `Nome_Cliente`, `Email`, `Tipo_Solicitacao`, `Valor`, `Data_Solicitacao`, `Status` |
| **Resultado** | Relatório de Processamento | `ID_Solicitacao`, `Nome_Cliente`, `Tipo_Solicitacao`, `Valor`, `Status_Processamento`, `Data_Hora_Processamento`, `Mensagem_Observacao` |
| **Erros** | Relatório de Auditoria | Colunas originais + `Motivo_Erro` |

---

## 🛠️ Tecnologias e Pacotes Utilizados

- **UiPath Studio** (Desenvolvimento RPA)
- **UiPath.GSuite.Activities** (Integração nativa com Google Sheets)
- **VB.NET** (Manipulação de DataTables, Strings e expressões ternárias)

---

## 🚀 Como Executar

1. Abra o projeto no **UiPath Studio**.
2. Certifique-se de que a conexão com a conta do Google Drive está ativa na atividade **Google Workspace Scope**.
3. Abra o arquivo **`Main.xaml`**.
4. Pressione **F5** (ou clique em **Run**).
5. Acompanhe os logs no painel **Output** e confira as abas **Resultado** e **Erros** na planilha Google.
