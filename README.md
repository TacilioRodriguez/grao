# Projeto de ETL de Logística com PySpark e Arquitetura Medallion

Este projeto demonstra um pipeline de dados completo (ETL) construído no ambiente Databricks utilizando PySpark. O objetivo é processar dados brutos de envios logísticos, estruturá-los seguindo as melhores práticas da Arquitetura Medallion (Bronze, Silver e Gold) e, ao final, gerar tabelas agregadas com insights de negócio.

## 🚀 Tecnologias Utilizadas

* **Cloud:** Databricks
* **Processamento de Dados:** Apache Spark (com a API PySpark)
* **Formato de Armazenamento:** Delta Lake
* **Arquitetura:** Medallion (Bronze, Silver e Gold)
* **Linguagem:** Python

## 🏛️ Arquitetura do Projeto

O pipeline foi estruturado em três camadas lógicas, garantindo governança, qualidade e escalabilidade dos dados.

### 🥉 Camada Bronze
A primeira camada armazena os dados brutos, exatamente como foram recebidos, servindo como uma fonte de verdade imutável.
* **Tabela:** `grao.bronze.grain_shipping`
    * **Origem:** Cópia fiel dos dados do arquivo `grain_logistic_shipping.csv`.
* **Tabela:** `grao.bronze.calendario`
    * **Origem:** Tabela de dimensão de tempo, gerada para cobrir o período de análise e facilitar a manipulação de datas.

### 🥈 Camada Silver
Nesta camada, os dados brutos são limpos, validados, enriquecidos e modelados em um esquema **Star Schema**, otimizado para consultas e análises.

* **Tabelas de Dimensão:**
    * `dim_localizacao`: Contém a lista de estados de destino dos envios.
    * `dim_cliente`: Descreve características dos clientes (neste caso, por gênero).
    * `dim_metodo_envio`: Detalha os atributos da operação de envio, como método, corredor de armazenagem e importância.
    * `dim_calendario`: A tabela da camada Bronze é utilizada como dimensão de tempo.

* **Tabela Fato:**
    * `fato_envios`: Tabela central que contém as métricas de cada envio (preço, peso, quantidade de itens, etc.) e as chaves estrangeiras (IDs) que a conectam a todas as tabelas de dimensão.

### 🥇 Camada Gold
A camada final, construída a partir da Silver. Ela contém dados agregados e prontos para o consumo por ferramentas de BI, dashboards e análises de negócio.

* **Tabelas Agregadas:**
    * `gold_vendas_por_destino`: Sumariza a receita total, o número de envios e o valor médio por estado.
    * `gold_performance_por_metodo_envio`: Analisa a eficiência de cada método de envio, calculando a taxa de entrega no prazo e a avaliação média dos clientes.
    * `gold_resumo_mensal_operacoes`: Apresenta uma visão mensal da evolução da receita, do volume de envios e do peso total transportado.

## 🌊 Fluxo do Pipeline de Dados

O processo de transformação dos dados seguiu as seguintes etapas:

1.  **Ingestão (Bronze):** Os dados brutos do CSV foram carregados e salvos como uma tabela Delta na camada Bronze, garantindo um ponto de partida confiável para todo o pipeline.

2.  **Limpeza e Preparação:** Os nomes das colunas foram padronizados (removendo acentos, caracteres especiais e convertendo para minúsculas) para facilitar a manipulação no PySpark.

3.  **Criação da Camada Silver:**
    * As tabelas de dimensão foram criadas extraindo-se os valores distintos das colunas categóricas da tabela Bronze e adicionando uma chave substituta (ID) para cada uma.
    * O desafio de converter as datas em formato de texto complexo (ex: "quarta-feira, 1 de junho de 2022") foi resolvido de forma robusta, utilizando um `JOIN` com a tabela `dim_calendario`.
    * A tabela `fato_envios` foi construída unindo a tabela Bronze com todas as dimensões, substituindo os valores de texto pelas suas respectivas chaves (IDs).

4.  **Criação da Camada Gold:**
    * A partir da união da tabela `fato_envios` com as dimensões, foram aplicadas operações de `groupBy` e agregações (`sum`, `avg`, `count`) para calcular as métricas de negócio e criar as tabelas da camada Gold.
    * As colunas de valor foram arredondadas para duas casas decimais para uma apresentação mais limpa.

## 📊 Insights Gerados

As tabelas da camada Gold foram projetadas para responder a perguntas de negócio cruciais, tais como:
* Quais são os estados mais e menos lucrativos?
* Qual parceiro logístico oferece o serviço mais pontual e com maior satisfação do cliente?
* O volume de vendas está crescendo ou diminuindo ao longo dos meses?
* Existe alguma correlação entre o método de envio e a avaliação do cliente?

## 🧑‍💻 Como Executar

Para replicar este projeto, é necessário um ambiente Databricks configurado. O pipeline pode ser executado através de notebooks, seguindo a ordem lógica das camadas:
1.  **Notebook 01_Bronze:** Ingestão do CSV e criação da tabela `grao.bronze.grain_shipping`. Criação da tabela `grao.bronze.calendario`.
2.  **Notebook 02_Silver:** Leitura das tabelas Bronze e criação de todas as tabelas de dimensão e da tabela fato.
3.  **Notebook 03_Gold:** Leitura das tabelas Silver e criação das tabelas agregadas para análise.

---
