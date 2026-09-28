# Catálogo de Dados

## 1. Visão Geral

Este documento apresenta o catálogo de dados do projeto **MVP – Engenharia de Dados**, desenvolvido no Databricks utilizando arquitetura Medallion.

O catálogo documenta as principais estruturas utilizadas no pipeline, apresentando:

* camada de dados;
* tabela ou estrutura;
* campo;
* descrição;
* tipo de dado;
* domínio ou unidade;
* origem/linhagem;
* finalidade analítica.

O fluxo dos dados é:

```text
Kaggle
   ↓
Staging
   ↓
Bronze
   ↓
Silver
   ↓
Gold Daily
   ↓
Gold Yearly
   ↓
Análises
```

---

# 2. Staging

## 2.1 Identificação

**Localização:**

`mvp_eng_dados.staging.gold_price_data`

**Arquivo:**

`Gold Futures Historical Data.csv`

**Origem:** Kaggle

**Granularidade:** uma observação por dia de negociação.

**Registros:** 4.997

**Campos:** 7

O arquivo é armazenado no Volume de Staging sem alteração dos dados originais. Na leitura inicial, as colunas são mantidas como `string`, sendo a tipagem realizada posteriormente na camada Silver.

## 2.2 Dicionário de campos

| Campo      | Tipo   | Descrição                                 | Domínio / Unidade                 | Origem |
| ---------- | ------ | ----------------------------------------- | --------------------------------- | ------ |
| `Date`     | string | Data da observação do preço do ouro       | Data no formato original da fonte | Kaggle |
| `Price`    | string | Preço de fechamento observado para o ouro | Valor numérico do preço           | Kaggle |
| `Open`     | string | Preço de abertura do período              | Valor numérico do preço           | Kaggle |
| `High`     | string | Maior preço observado no período          | Valor numérico do preço           | Kaggle |
| `Low`      | string | Menor preço observado no período          | Valor numérico do preço           | Kaggle |
| `Vol.`     | string | Volume negociado no período               | Volume de negociação              | Kaggle |
| `Change %` | string | Variação percentual informada pela fonte  | Percentual (%)                    | Kaggle |

---

# 3. Bronze

## 3.1 Identificação

**Tabela:**

`mvp_eng_dados.bronze.gold_price`

**Granularidade:** uma observação por dia de negociação.

**Registros:** 4.997

**Campos:** 9

A camada Bronze representa a primeira persistência estruturada dos dados. Os campos recebem nomes padronizados com o sufixo `_raw`, mantendo a característica de dados brutos. Também são adicionados campos técnicos de rastreabilidade da ingestão.

## 3.2 Dicionário de campos

| Campo                  | Tipo      | Descrição                                  | Domínio / Unidade                       | Origem               |
| ---------------------- | --------- | ------------------------------------------ | --------------------------------------- | -------------------- |
| `date_raw`             | string    | Data original da observação                | Data em formato original                | Staging              |
| `price_raw`            | string    | Preço original de fechamento               | Valor numérico como recebido            | Staging              |
| `open_raw`             | string    | Preço original de abertura                 | Valor numérico como recebido            | Staging              |
| `high_raw`             | string    | Maior preço original observado             | Valor numérico como recebido            | Staging              |
| `low_raw`              | string    | Menor preço original observado             | Valor numérico como recebido            | Staging              |
| `volume_raw`           | string    | Volume original de negociação              | Volume como recebido                    | Staging              |
| `change_pct_raw`       | string    | Variação percentual original               | Percentual (%) como recebido            | Staging              |
| `_ingestion_timestamp` | timestamp | Data e hora em que o registro foi ingerido | Timestamp                               | Processo de ingestão |
| `_source`              | string    | Identificação da fonte dos dados           | `Kaggle - Gold Futures Historical Data` | Processo de ingestão |

---

# 4. Silver

## 4.1 Identificação

**Tabela:**

`mvp_eng_dados.silver.gold_price`

**Granularidade:** uma observação por dia de negociação.

**Registros:** 4.997

**Campos:** 11

A camada Silver realiza a padronização dos dados, conversão dos tipos, tratamento da data e criação das dimensões temporais `year` e `month`. Os campos de preço são convertidos para `double`, volume para `long` e variação percentual para `double`.

## 4.2 Dicionário de campos

| Campo                  | Tipo      | Descrição                                | Domínio / Unidade      | Origem / Transformação                     |
| ---------------------- | --------- | ---------------------------------------- | ---------------------- | ------------------------------------------ |
| `date`                 | date      | Data da observação                       | Data                   | `date_raw`, convertida para date           |
| `year`                 | integer   | Ano da observação                        | Ano calendário         | Derivado de `date`                         |
| `month`                | integer   | Mês da observação                        | 1 a 12                 | Derivado de `date`                         |
| `price`                | double    | Preço de fechamento                      | Valor do preço         | `price_raw`, convertido para numérico      |
| `open_price`           | double    | Preço de abertura                        | Valor do preço         | `open_raw`, convertido para numérico       |
| `high_price`           | double    | Maior preço observado                    | Valor do preço         | `high_raw`, convertido para numérico       |
| `low_price`            | double    | Menor preço observado                    | Valor do preço         | `low_raw`, convertido para numérico        |
| `volume`               | long      | Volume negociado                         | Quantidade/volume      | `volume_raw`, convertido para inteiro      |
| `change_pct`           | double    | Variação percentual informada pela fonte | Percentual (%)         | `change_pct_raw`, convertido para numérico |
| `_ingestion_timestamp` | timestamp | Data e hora da ingestão                  | Timestamp              | Herdado da Bronze                          |
| `_source`              | string    | Fonte original do dado                   | Identificação da fonte | Herdado da Bronze                          |

### Principais transformações

As principais transformações realizadas na camada Silver foram:

1. Padronização do campo de data para o tipo `date`.
2. Conversão dos campos de preço para `double`.
3. Conversão do volume para `long`.
4. Conversão da variação percentual para `double`.
5. Criação do campo `year`.
6. Criação do campo `month`.
7. Preservação dos campos técnicos de rastreabilidade.

Essas transformações são realizadas diretamente no notebook do projeto.

---

# 5. Gold – Dados Diários

## 5.1 Identificação

**Tabela:**

`mvp_eng_dados.gold.gold_price_daily`

**Granularidade:** uma observação por dia de negociação.

**Registros:** 4.997

**Campos:** 13

A tabela Gold diária é construída a partir da Silver e adiciona métricas derivadas utilizadas nas análises de comportamento diário do preço do ouro.

## 5.2 Dicionário de campos

| Campo              | Tipo    | Descrição                                                            | Domínio / Unidade | Origem / Transformação               |
| ------------------ | ------- | -------------------------------------------------------------------- | ----------------- | ------------------------------------ |
| `date`             | date    | Data da observação                                                   | Data              | Silver                               |
| `year`             | integer | Ano da observação                                                    | Ano calendário    | Silver                               |
| `month`            | integer | Mês da observação                                                    | 1 a 12            | Silver                               |
| `price`            | double  | Preço de fechamento                                                  | Valor do preço    | Silver                               |
| `open_price`       | double  | Preço de abertura                                                    | Valor do preço    | Silver                               |
| `high_price`       | double  | Maior preço do dia                                                   | Valor do preço    | Silver                               |
| `low_price`        | double  | Menor preço do dia                                                   | Valor do preço    | Silver                               |
| `volume`           | long    | Volume negociado                                                     | Quantidade/volume | Silver                               |
| `change_pct`       | double  | Variação percentual informada pela fonte                             | Percentual (%)    | Silver                               |
| `previous_price`   | double  | Preço de fechamento da observação anterior                           | Valor do preço    | Calculado com base na série temporal |
| `daily_return_pct` | double  | Retorno diário calculado a partir do preço atual e do preço anterior | Percentual (%)    | Calculado                            |
| `daily_range`      | double  | Amplitude diária entre o maior e o menor preço                       | Unidade do preço  | `high_price - low_price`             |
| `daily_range_pct`  | double  | Amplitude diária em relação ao preço                                 | Percentual (%)    | Amplitude diária normalizada         |

### Observação

O primeiro registro da série possui `previous_price` e `daily_return_pct` nulos, pois não existe uma observação anterior disponível para o cálculo.

---

# 6. Gold – Dados Anuais

## 6.1 Identificação

**Tabela:**

`mvp_eng_dados.gold.gold_price_yearly`

**Granularidade:** uma observação por ano.

**Registros:** 22

**Campos:** 10

A tabela Gold anual é construída a partir da Gold diária e consolida os principais indicadores utilizados para responder às perguntas de negócio.

## 6.2 Dicionário de campos

| Campo                      | Tipo    | Descrição                                           | Domínio / Unidade    | Origem / Transformação                 |
| -------------------------- | ------- | --------------------------------------------------- | -------------------- | -------------------------------------- |
| `year`                     | integer | Ano de referência                                   | Ano calendário       | Gold Daily                             |
| `first_price`              | double  | Primeiro preço observado no ano                     | Valor do preço       | Primeiro preço cronológico do ano      |
| `last_price`               | double  | Último preço observado no ano                       | Valor do preço       | Último preço cronológico do ano        |
| `average_price`            | double  | Preço médio observado no ano                        | Valor médio do preço | Média anual                            |
| `min_price`                | double  | Menor preço observado no ano                        | Valor do preço       | Mínimo anual                           |
| `max_price`                | double  | Maior preço observado no ano                        | Valor do preço       | Máximo anual                           |
| `annual_return_pct`        | double  | Retorno entre o primeiro e o último preço do ano    | Percentual (%)       | `(last_price / first_price - 1) × 100` |
| `average_daily_return_pct` | double  | Média dos retornos diários do ano                   | Percentual (%)       | Média de `daily_return_pct`            |
| `volatility_pct`           | double  | Desvio-padrão dos retornos diários                  | Percentual (%)       | Desvio-padrão de `daily_return_pct`    |
| `trading_days`             | long    | Quantidade de observações/dias de negociação no ano | Número de dias       | Contagem dos registros                 |

A fórmula utilizada para o retorno anual está implementada no notebook como:

```text
((last_price / first_price) - 1) × 100
```

A volatilidade anual é calculada a partir do desvio-padrão dos retornos diários.

---

# 7. Linhagem dos Dados

A linhagem principal dos dados pode ser representada da seguinte forma:

```text
Kaggle
│
│  Gold Futures Historical Data.csv
↓
STAGING
│
│  Dados originais
↓
BRONZE
│
│  Padronização de nomes + metadados técnicos
↓
SILVER
│
│  Tipagem + tratamento de datas + year/month
↓
GOLD DAILY
│
│  Métricas diárias derivadas
│
├───────────────┐
│               │
│               ↓
│          Análises diárias
│
↓
GOLD YEARLY
│
│  Agregações anuais
│
├── average_price
├── annual_return_pct
├── volatility_pct
└── trading_days
│
↓
ANÁLISES DE NEGÓCIO
```

## 7.1 Resumo da linhagem por tabela

| Estrutura   | Origem      | Principais transformações                                          |
| ----------- | ----------- | ------------------------------------------------------------------ |
| Staging     | Kaggle      | Nenhuma                                                            |
| Bronze      | Staging     | Renomeação dos campos + metadados de ingestão                      |
| Silver      | Bronze      | Tipagem, tratamento de datas e criação de `year`/`month`           |
| Gold Daily  | Silver      | Cálculo de preço anterior, retorno diário e amplitude              |
| Gold Yearly | Gold Daily  | Agregações e indicadores anuais                                    |
| Análises    | Gold Yearly | Indicadores e visualizações para responder às perguntas de negócio |

---

# 8. Regras e Observações de Qualidade

A validação realizada no pipeline identificou:

* **4.997 registros** na base;
* **4.997 datas distintas**;
* **0 registros duplicados por data**;
* **0 registros com valores negativos**;
* **4 registros com relações de preço consideradas inconsistentes** entre os campos de preço.

As quatro inconsistências identificadas foram preservadas para manter os valores originais da fonte. Não foram realizadas correções arbitrárias nos dados.

O tratamento e as validações de qualidade estão documentados e implementados no notebook principal do projeto.

---

# 9. Referência Técnica

O catálogo foi elaborado com base nas estruturas efetivamente criadas e persistidas no notebook:

**[PUC_MVP_Engenharia_de_Dados.ipynb](./PUC_MVP_Engenharia_de_Dados.ipynb)**

O notebook contém as etapas de ingestão, criação das camadas Bronze, Silver e Gold, validação da qualidade dos dados e geração das análises.

