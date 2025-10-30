

# Agente de Relatórios com OpenAI (Desafio Indicium AI Engineer)

Este repositório contém a criação de um agente ReAct usando a API da OpenAI para gerar relatórios. O relatório envolve a criação de métricas e a observação de tendências usando o banco de dados da Síndrome Respiratória Aguda Grave (SRAG) de 2019 a 2025, disponível publicamente no [Opendatasus](https://opendatasus.saude.gov.br/dataset/srag-2021-a-2024).

Este trabalho faz parte do desafio para a certificação de AI Engineer da Indicium.

---

## 1. Descrição do Projeto

Este projeto tem o objetivo de criar uma solução baseada em dados que auxilia profissionais da saúde a terem um entendimento maior sobre certas doenças, visando simplificar a visualização da severidade e surto de determinadas doenças em tempo real.

## 2. Ferramentas e Tecnologias

* **Fonte (Banco de Dados):** 2025 - Banco vivo 13/10/2025 - CSV
* **Plataforma de Dados:** Databricks (versão Community/Free)
* **LLM (Cérebro):** GPT-4 (via API da OpenAI)
* **Tool (SQL):** LangChain SQLDatabase
* **Tool (Web Search):** SerpAPI
* **Agente (Orquestrador):** LangGraph (para um agente ReAct cíclico)
* **Observação:** Os prompts foram escritos em inglês para otimizar o raciocínio e a performance do agente.

---

## 3. Arquitetura e Funcionamento

Abaixo está o diagrama conceitual da solução, seguido por uma descrição detalhada do fluxo de execução.

<img width="487" height="464" alt="image" src="https://github.com/user-attachments/assets/448b8241-e97d-45ea-ae30-69d5487f9996" />



### Fluxo Resumido

1.  Ingestão dos dados no Databricks.
2.  Limpeza e criação da tabela `srag_analysis`.
3.  Ativação do ambiente e carregamento das chaves de API.
4.  Criação de Widgets e arquivo `.env`.
5.  Ativação do agente.
6.  Pergunta do usuário.
7.  Utilização das *Tools* (SQL ou Web Search) pelo agente.
8.  Geração da resposta baseada nos dados coletados.
9.  Geração de gráficos (quando solicitado).
10. Compilação do relatório final.

### Funcionamento Detalhado do Agente

O fluxo de interação do usuário com o agente segue a sequência:

1.  O **Usuário** envia um **Prompt** (pergunta) para o sistema.
2.  O **Agente Principal** (LangGraph) recebe o prompt e o encaminha, junto com a lista de ferramentas disponíveis, para **O Cérebro (LLM - GPT-4)**.
3.  O **LLM** analisa a intenção da pergunta e decide qual ferramenta é a mais adequada para respondê-la, informando sua decisão ao Agente.
4.  O **Agente** invoca a ferramenta selecionada pelo LLM.

#### Cenário A: Consulta ao Banco de Dados (SQLDatabase Tool)
* **5A.** A Ferramenta SQL é ativada. Ela envia a pergunta do usuário para o **LLM (Cérebro)** com a instrução de traduzi-la.
* **6A.** O **LLM** gera uma consulta **SQL** precisa e a retorna para a ferramenta.
* **7A.** A ferramenta executa a consulta SQL diretamente no **Banco de Dados (Databricks)**.
* **8A.** O Banco de Dados retorna os **Resultados da Tabela** (dados).
* **9A.** A ferramenta envia os dados encontrados de volta ao Agente como uma "Observação".

#### Cenário B: Busca na Web (SerpAPI Tool)
* **5B.** A Ferramenta SerpAPI é ativada.
* **6B.** A ferramenta executa uma consulta na **SerpAPI** para buscar informações nas **Fontes de Notícias (Internet)**.
* **7B.** Os **Resultados** da busca são retornados para a ferramenta.
* **8B.** A ferramenta envia os links e snippets encontrados de volta ao Agente como uma "Observação".

#### Conclusão do Fluxo
* **Geração da Resposta:** O Agente Principal coleta todas as "Observações" (sejam os dados do banco ou os resultados da web) e as envia como **Contexto Final** para **O Cérebro (LLM)**.
* O **LLM** sintetiza todas as informações de contexto e gera uma **Resposta Final** coesa em linguagem natural para o usuário.

---

## 4. Como Usar (Configuração e Execução)

O projeto é dividido em dois notebooks principais que devem ser executados em ordem.

1.  `config_desafio_fai.ipynb`
2.  `fai_desafio.ipynb`

### Pré-requisito

> **IMPORTANTE:** ANTES DE TUDO, VOCÊ DEVERÁ CRIAR UM **CATALOG** E UM **SCHEMA** PELA INTERFACE DE USUÁRIO (UI) DO DATABRICKS.

---

### Passo 1: Notebook `config_desafio_fai`

Este notebook é responsável pela ingestão dos dados, limpeza, análise exploratória (PLN) e criação do arquivo `.env` no volume do Databricks.

**Célula 1: Widgets**
Edite os valores padrão para que correspondam ao seu ambiente no Databricks.

```python
# Valores padrão são apenas sugestões, podem ser alterados ao rodar o notebook
dbutils.widgets.text("catalog_name", "edite_nome_aqui", "Nome do Catálogo UC")
dbutils.widgets.text("schema_name", "edite_nome_aqui", "Nome do Schema UC")
dbutils.widgets.text("volume_name", "edite_nome_aqui", "Nome do Volume UC") # Vamos criar mais pra frente, mas você já pode adicionar o nome que ele terá
dbutils.widgets.text("table_name", "srag_analysis", "Nome da Tabela") 
dbutils.widgets.text("env_file_name", ".env", "Nome do Arquivo de Configuração (.env)")
```

**Descrição das Células:**
* **Célula 2:** Cria a tabela `srag_analysis` fazendo a limpeza, adição e seleção de colunas do CSV bruto.
* **Célula 3:** Imports de bibliotecas para PLN (Processamento de Linguagem Natural).
* **Célula 4:** Limpeza de dados textuais e verificação da frequência de palavras.
* **Célula 5:** Construção de bigramas para analisar a frequência de termos compostos.
* **Célula 6:** Plot (gráfico) dos 20 bigramas mais frequentes.
* **Célula 7:** Visualização da tabela final.
* **Célula 8:** Criação do **Volume** (usando o `volume_name` definido no widget) para preparar o ambiente para a criação do `.env`.
* **Célula 9:** Criação do arquivo `.env`.

#### Como configurar o arquivo `.env` (Célula 9)

Esta célula grava suas chaves de API secretas em um arquivo `.env` dentro do volume do Databricks.

**CASO NÃO SAIBA ONDE PEGAR O TOKEN, HOSTNAME E PATH DO DATABRICKS, SIGA OS PASSOS:**

1.  **Token (DATABRICKS_TOKEN)**
    * Clique no seu ícone de perfil no canto superior direito.
    * Clique em **Settings**.
    * Em **User Settings**, clique em **Developer**.
    * Crie seu *Personal Access Token* e salve-o.
2.  **Hostname e Path (SERVER_HOSTNAME e HTTP_PATH)**
    * No menu do lado esquerdo, clique em **SQL Warehouses**.
    * Clique no *Warehouse* que você está usando (ex: *Serverless Starter Warehouse*).
    * Clique em **Connection details**.
    * Copie os dois caminhos que aparecem (`Host` e `HTTP path`).

**AGORA**, informe suas variáveis de ambiente no bloco de código da Célula 9:

```python
# (Exemplo de como a string deve ser preenchida)
DATABRICKS_TOKEN=INSIRA_SEU_TOKEN_DATABRICKS_AQUI
SERVER_HOSTNAME=INSIRA_SEU_HOSTNAME_DATABRICKS_AQUI 
HTTP_PATH=INSIRA_SEU_HTTP_PATH_WAREHOUSE_AQUI
SERPAPI_API_KEY=INSIRA_SUA_CHAVE_SERPAPI_AQUI
OPENAI_API_KEY=INSIRA_SUA_CHAVE_OPENAI_AQUI
```

Execute a célula para salvar o arquivo.

---

### Passo 2: Notebook `fai_desafio`

Este notebook carrega o ambiente, constrói o agente e executa as lógicas de perguntas e respostas.

**Célula 1: Widgets**
Preencha com **os mesmos valores** que você usou no Notebook 1.

```python
# Valores padrão são apenas sugestões, podem ser alterados ao rodar o notebook
dbutils.widgets.text("catalog_name", "edite_nome_aqui", "Nome do Catálogo UC")
dbutils.widgets.text("schema_name", "edite_nome_aqui", "Nome do Schema UC")
dbutils.widgets.text("volume_name", "edite_nome_aqui", "Nome do Volume UC")
dbutils.widgets.text("table_name", "edite_nome_aqui", "Nome da Tabela")
dbutils.widgets.text("env_file_name", "edite_nome_aqui", "Nome do Arquivo de Configuração (.env)")
```

**Descrição das Células:**

* **Célula 2:** Importa todas as bibliotecas necessárias (LangChain, Databricks, etc.).
* **Célula 3:** Reinicia o Kernel (necessário para que as bibliotecas recém-instaladas sejam reconhecidas).
* **Célula 4:** Carrega as variáveis de ambiente (suas chaves de API) a partir do arquivo `.env` salvo no volume.
* **Célula 5:** Configura o LLM (modelo `gpt-4`).
* **Célula 6:** Conecta à tabela `srag_analysis` no Databricks.
* **Célula 7:** Configura a **Tool SQLDatabase**, que usa o próprio LLM (GPT-4) para traduzir perguntas em consultas SQL.
* **Célula 8:** Configura a **Tool SerpAPI** (Web Search).
* **Célula 9:** Combina as duas ferramentas em uma lista para o agente.
* **Célula 10:** Cria o agente ReAct utilizando **LangGraph**.
* **Célula 11:** Visualiza o grafo (fluxograma) do agente recém-criado.
* **Célula 12:** Define o `system_prompt` (Contexto e Regras) para guiar o raciocínio do agente.
* **Células 13 a 21:** Perguntas feitas ao agente.
    * A variável `user_question` é usada para fazer a pergunta.
    * Dependendo do prompt, o agente decidirá se deve consultar o banco de dados (Tool A) ou a internet (Tool B).
    * As respostas são guardadas em suas respectivas variáveis (ex: `response_1`, `response_2`) para auxiliar na criação do relatório final.
    * O `print(messages)` ao final de cada célula é opcional e mostra o "raciocínio" passo a passo do agente, sendo excelente para auditoria e debug.
* **Célula 22:** Query SQL manual para agrupar registros de casos diários e dos últimos 12 meses (preparação para o gráfico).
* **Célula 23 e 24:** Checagem e conferência dos dados diários.
* **Célula 25:** Plotagem dos gráficos de tendência.
* **Célula 26:** **Geração do Relatório Final**, utilizando as variáveis de resposta (ex: `response_1`, `response_2`, etc.) salvas nas células anteriores para compilar o texto.

---

## 5. Autor
* **Repo owner:** Jéssica Figueiredo
* **GitHub:** [github.com/Jessicarla](https://github.com/Jessicarla)
