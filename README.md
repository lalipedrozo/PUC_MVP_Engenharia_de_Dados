# MVP – Engenharia de Dados

## Introdução

Este projeto foi desenvolvido como parte do MVP da disciplina de Engenharia de Dados da Pós-Graduação em Data Science & Analytics da PUC-Rio.

O projeto apresenta a construção de um pipeline completo de dados utilizando **Databricks** e arquitetura **Medallion**, contemplando as etapas de ingestão, armazenamento, transformação, validação de qualidade, modelagem e análise de dados.

O conjunto de dados utilizado contém informações históricas de preços do ouro e permite analisar a evolução dos preços ao longo do tempo, os retornos anuais e a volatilidade do ativo.

---

# 1. Contexto de Negócios e Perguntas

## 1.1 Contexto

O ouro é um ativo amplamente utilizado como instrumento de investimento e reserva de valor, sendo acompanhado por investidores, instituições financeiras e analistas de mercado.

A análise histórica de seus preços permite identificar padrões de valorização e desvalorização, períodos de maior volatilidade e mudanças relevantes no comportamento do ativo ao longo do tempo.

Neste projeto, foi utilizado um conjunto de dados históricos contendo preços diários do ouro, possibilitando a construção de indicadores e análises a partir dos dados brutos.

## 1.2 Objetivo

O objetivo do projeto é construir um pipeline de dados completo e estruturado para processamento de informações históricas do preço do ouro, desde a ingestão dos dados brutos até a disponibilização de tabelas analíticas na camada Gold.

A partir dos dados tratados, o projeto busca responder perguntas relacionadas à evolução dos preços, aos retornos anuais e à volatilidade do ouro.

## 1.3 Perguntas de negócio

O projeto busca responder às seguintes perguntas:

1. **Como evoluiu o preço médio anual do ouro ao longo do período analisado?**

2. **Quais foram os anos de maior e menor retorno anual do ouro?**

3. **Quais períodos apresentaram maior volatilidade?**

4. **Existem limitações ou inconsistências nos dados que devem ser consideradas na interpretação dos resultados?**

## 1.4 Fonte dos dados

Os dados utilizados foram obtidos a partir de um dataset disponibilizado no **Kaggle**, contendo informações históricas de preços do ouro.

A base apresenta observações diárias e variáveis relacionadas aos preços de abertura, máxima, mínima e fechamento.

A fonte original dos dados pode ser consultada no Kaggle.

## 1.5 Licença dos dados

A página do dataset no Kaggle informa a licença **CC0: Public Domain**, permitindo o uso dos dados sem restrições de copyright, conforme os termos apresentados na fonte.

---

# 2. Carga dos Dados

## 2.1 Processo de coleta

O arquivo de dados foi obtido a partir do Kaggle e disponibilizado no ambiente **Databricks**.

O arquivo bruto foi armazenado inicialmente na camada de **Staging**, sem alterações nos dados originais.

O processo foi organizado para separar o arquivo de origem das etapas posteriores de transformação, permitindo preservar os dados brutos e manter a rastreabilidade do pipeline.

## 2.2 Staging

O arquivo original foi armazenado em um **Volume do Unity Catalog**, no seguinte caminho:

`mvp_eng_dados.staging.gold_price_data`

A camada de Staging representa o ponto inicial do pipeline e tem como objetivo armazenar os dados recebidos da fonte antes da aplicação das transformações.

O arquivo bruto não é utilizado diretamente para as análises finais. A partir dele são construídas as tabelas das camadas Bronze, Silver e Gold.

## 2.3 Armazenamento na nuvem

Todo o processamento foi realizado no **Databricks**, utilizando armazenamento e tabelas persistidas no **Unity Catalog**.

A estrutura criada no ambiente é:

```text
mvp_eng_dados
│
├── staging
│   └── gold_price_data
│
├── bronze
│   └── gold_price
│
├── silver
│   └── gold_price
│
└── gold
    ├── gold_price_daily
    └── gold_price_yearly
```

Essa estrutura garante a separação entre os dados brutos, dados tratados e dados preparados para análise.

---

# 3. Modelagem e Catálogo de Dados

## 3.1 Arquitetura Medallion

O projeto utiliza a arquitetura **Medallion**, organizando o fluxo de dados em diferentes camadas de processamento:

```text
Kaggle
   ↓
Staging
   ↓
Bronze
   ↓
Silver
   ↓
Data Quality
   ↓
Gold
   ├── gold_price_daily
   └── gold_price_yearly
   ↓
Análises
```

Cada camada possui uma responsabilidade específica:

* **Staging:** armazenamento do arquivo bruto recebido da fonte.
* **Bronze:** ingestão dos dados para uma tabela estruturada, preservando a informação original.
* **Silver:** tratamento, padronização e preparação dos dados para utilização analítica.
* **Data Quality:** validação da qualidade, completude, unicidade e consistência dos dados.
* **Gold:** criação das tabelas finais destinadas à análise.
* **Análises:** utilização das tabelas Gold para geração dos indicadores e respostas às perguntas de negócio.

## 3.2 Estrutura das camadas

### Staging

Armazena o arquivo original recebido da fonte.

### Bronze

Tabela:

`mvp_eng_dados.bronze.gold_price`

A camada Bronze representa a primeira persistência estruturada dos dados no ambiente analítico.

### Silver

Tabela:

`mvp_eng_dados.silver.gold_price`

A camada Silver contém os dados tratados e padronizados, preparados para as validações e agregações posteriores.

### Gold

A camada Gold contém duas tabelas principais:

**Tabela diária**

`mvp_eng_dados.gold.gold_price_daily`

Essa tabela mantém os dados em granularidade diária e inclui métricas derivadas, como:

* preço anterior;
* retorno diário;
* amplitude diária;
* amplitude percentual diária.

**Tabela anual**

`mvp_eng_dados.gold.gold_price_yearly`

Essa tabela apresenta os dados agregados por ano, permitindo análises como:

* primeiro preço do ano;
* último preço do ano;
* preço médio anual;
* retorno anual;
* retorno diário médio;
* volatilidade;
* quantidade de dias de negociação.

## 3.3 Tabelas

| Camada | Tabela                                 | Objetivo                                  |
| ------ | -------------------------------------- | ----------------------------------------- |
| Bronze | `mvp_eng_dados.bronze.gold_price`      | Persistência estruturada dos dados brutos |
| Silver | `mvp_eng_dados.silver.gold_price`      | Dados tratados e padronizados             |
| Gold   | `mvp_eng_dados.gold.gold_price_daily`  | Dados diários e métricas derivadas        |
| Gold   | `mvp_eng_dados.gold.gold_price_yearly` | Agregações e indicadores anuais           |

## 3.4 Catálogo de Dados

O detalhamento dos campos, tipos de dados, descrições, domínios e linhagem está disponível no arquivo:

**[Catálogo de Dados](./Catalogo_de_Dados.md)**

O catálogo documenta as principais tabelas utilizadas no pipeline e permite identificar a origem e a finalidade de cada campo.

---

# 4. Pipeline de Dados

## 4.1 Arquitetura do pipeline

O pipeline desenvolvido segue o fluxo:

```text
Fonte de dados – Kaggle
        ↓
Staging
        ↓
Bronze
        ↓
Silver
        ↓
Validações de Data Quality
        ↓
Gold Daily
        ↓
Gold Yearly
        ↓
Análises
```

O pipeline foi implementado em um notebook Databricks, disponível neste repositório:

**[PUC_MVP_Engenharia_de_Dados.ipynb](./PUC_MVP_Engenharia_de_Dados.ipynb)**

## 4.2 Bronze

Na camada Bronze, os dados do arquivo original são carregados para uma tabela persistida no Unity Catalog.

O objetivo dessa etapa é garantir uma primeira camada estruturada e persistente, mantendo a proximidade com os dados de origem.

Tabela:

`mvp_eng_dados.bronze.gold_price`

## 4.3 Silver

Na camada Silver são realizadas as transformações necessárias para preparar os dados para utilização analítica.

Entre as etapas realizadas estão:

* padronização dos tipos de dados;
* tratamento das colunas de data;
* organização dos campos de preço;
* preparação dos dados para cálculos derivados;
* persistência da tabela tratada.

Tabela:

`mvp_eng_dados.silver.gold_price`

## 4.4 Gold

A camada Gold concentra os dados preparados para análise.

Foram criadas duas visões de granularidade:

### Gold Daily

`mvp_eng_dados.gold.gold_price_daily`

Mantém a granularidade diária e acrescenta métricas derivadas, incluindo retorno diário e amplitude dos preços.

### Gold Yearly

`mvp_eng_dados.gold.gold_price_yearly`

Agrega os dados por ano e disponibiliza indicadores utilizados diretamente nas análises de negócio.

## 4.5 Execução e persistência

O notebook foi executado integralmente no ambiente Databricks, desde a leitura dos dados de Staging até a criação das tabelas Gold e execução das análises.

As tabelas utilizadas no projeto são persistidas no Unity Catalog, permitindo que os dados tratados sejam armazenados e reutilizados independentemente da execução de cada etapa intermediária do notebook.

---

# 5. Qualidade de Dados

## 5.1 Completude

Foi realizada uma verificação de valores nulos nas principais colunas utilizadas no pipeline.

Os dados utilizados nas análises apresentaram completude adequada, não sendo identificados valores nulos nas colunas relevantes da base tratada.

## 5.2 Unicidade

Foi realizada uma validação de registros duplicados.

Não foram identificados registros duplicados que comprometessem a análise da série histórica.

## 5.3 Consistência

Foram realizadas verificações de consistência relacionadas aos valores de preço, incluindo:

* preços negativos;
* relação entre preços de abertura, máxima, mínima e fechamento;
* consistência dos registros ao longo da série temporal.

Não foram encontrados preços negativos.

## 5.4 Acurácia

Durante a validação foram identificados **4 registros com pequenas inconsistências entre os valores de abertura, máxima, mínima e fechamento**.

As inconsistências foram consideradas pequenas e não foram realizadas alterações nos valores originais, evitando a criação de dados artificiais.

## 5.5 Tratamento das inconsistências

A decisão adotada foi **preservar os valores originais da fonte** e documentar as inconsistências identificadas.

Essa abordagem mantém a rastreabilidade dos dados e evita alterações arbitrárias na informação original.

As inconsistências foram consideradas durante a interpretação dos resultados, especialmente na avaliação da qualidade e das limitações da base.

---

# 6. Análise de Dados

A análise foi realizada a partir das tabelas da camada Gold.

## 6.1 Evolução do preço

A análise do preço médio anual mostra uma tendência geral de valorização do ouro ao longo do período analisado, embora tenham ocorrido períodos de queda.

Entre os principais movimentos observados estão:

* queda relevante entre **2013 e 2015**;
* retomada da tendência de valorização a partir de **2019**;
* aceleração mais significativa do preço médio nos anos de **2024 e 2025**.

O gráfico de evolução do preço médio anual está apresentado no notebook.

## 6.2 Retorno anual

O retorno anual foi calculado a partir da variação entre o primeiro e o último preço de cada ano.

Os principais resultados observados foram:

* **2025:** maior retorno anual observado, de aproximadamente **+64,34%**;
* **2013:** menor retorno anual observado, de aproximadamente **−28,81%**;
* **2026:** retorno de aproximadamente **+18,26%**, porém o ano ainda está incompleto na base analisada.

Portanto, o resultado de 2026 deve ser interpretado com cautela, pois representa apenas parte do ano.

## 6.3 Volatilidade

A volatilidade anual foi utilizada para avaliar a dispersão dos retornos diários ao longo de cada ano.

Entre os anos completos, destacam-se:

* **2008:** aproximadamente **1,92%**;
* **2006:** aproximadamente **1,52%**;
* **2025:** aproximadamente **1,42%**;
* **2013:** aproximadamente **1,40%**;
* **2009:** aproximadamente **1,40%**.

O ano de **2026 apresentou volatilidade de aproximadamente 2,63%**, porém possui apenas parte dos dias de negociação do ano e, portanto, não deve ser comparado diretamente com anos completos sem essa ressalva.

## 6.4 Principais conclusões

A análise realizada permite concluir que:

1. O preço médio anual do ouro apresentou uma tendência geral de valorização no período analisado, com períodos intermediários de queda.

2. O ano de 2025 apresentou o maior retorno anual observado na base, enquanto 2013 apresentou o menor.

3. A volatilidade varia significativamente entre os anos, sendo importante considerar a quantidade de dias de negociação ao comparar períodos.

4. Os resultados de 2026 devem ser interpretados como dados parciais, pois o ano ainda não está completo na base utilizada.

5. A qualidade dos dados foi considerada adequada para a análise proposta, embora tenham sido identificadas quatro pequenas inconsistências entre campos de preço, que foram preservadas e documentadas.

---

# 7. Autoavaliação

O principal objetivo deste projeto foi desenvolver, na prática, um pipeline de engenharia de dados completo, passando por ingestão, armazenamento, transformação, validação de qualidade, modelagem e análise.

Durante o desenvolvimento, um dos principais desafios foi estruturar corretamente o fluxo entre as diferentes camadas da arquitetura Medallion e garantir que cada etapa tivesse uma responsabilidade clara.

Outro desafio foi trabalhar com a qualidade dos dados. Além de verificar valores nulos e duplicidades, foi necessário analisar a consistência entre os campos de preço e tomar uma decisão sobre como tratar os registros que apresentavam pequenas inconsistências.

A utilização do Databricks também permitiu compreender melhor a diferença entre armazenar dados, transformá-los e disponibilizá-los de forma persistente para consumo analítico.

Como evolução futura, o projeto poderia incluir:

* automatização da execução do pipeline;
* criação de monitoramento recorrente de qualidade dos dados;
* inclusão de novas fontes de dados;
* enriquecimento da análise com variáveis macroeconômicas;
* criação de dashboards para acompanhamento dos indicadores;
* implementação de análises preditivas ou modelos de séries temporais.

De forma geral, o projeto permitiu consolidar conceitos de engenharia de dados aplicados a um fluxo completo, desde a fonte dos dados até a geração de informações analíticas para apoio à tomada de decisão.
