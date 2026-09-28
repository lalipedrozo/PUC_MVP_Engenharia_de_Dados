# Autoavaliação

## 1. Objetivos do projeto

O principal objetivo deste MVP foi desenvolver, na prática, um pipeline completo de Engenharia de Dados em ambiente de nuvem, desde a ingestão dos dados brutos até a disponibilização de dados tratados e preparados para análise.

Ao longo do projeto, busquei aplicar os principais conceitos trabalhados na disciplina, especialmente:

* utilização de um ambiente de processamento em nuvem;
* organização dos dados segundo a arquitetura Medallion;
* separação entre as camadas Staging, Bronze, Silver e Gold;
* persistência dos dados utilizando o Unity Catalog;
* transformação e padronização dos dados;
* realização de validações de qualidade;
* criação de métricas derivadas;
* construção de tabelas analíticas;
* documentação da estrutura e da linhagem dos dados;
* disponibilização do código em um repositório público no GitHub.

Considero que os objetivos principais foram atingidos. O pipeline foi desenvolvido e executado integralmente no Databricks, desde a carga dos dados até a criação das tabelas Gold e das análises finais.

---

## 2. Principais dificuldades

Uma das principais dificuldades foi desenvolver o projeto inicialmente do zero no Databricks e compreender, na prática, como estruturar corretamente um pipeline utilizando as diferentes camadas da arquitetura Medallion.

Foi necessário entender a função de cada camada e evitar que transformações ou análises fossem realizadas em uma etapa inadequada do pipeline. A definição do fluxo **Staging → Bronze → Silver → Data Quality → Gold** foi importante para organizar o processamento e manter a rastreabilidade dos dados.

Outro desafio foi trabalhar com os dados da fonte original. Alguns campos numéricos estavam armazenados como texto e foi necessário realizar as conversões de tipos antes que os dados pudessem ser utilizados corretamente nas análises.

A etapa de qualidade dos dados também exigiu atenção. Embora a base não apresentasse problemas de completude ou duplicidade, foram identificadas quatro pequenas inconsistências entre campos de preço. Foi necessário analisar o problema e decidir como tratá-lo. A opção adotada foi preservar os dados originais e documentar as inconsistências, evitando alterações arbitrárias na fonte.

Outro aprendizado importante ocorreu durante a preparação do notebook para publicação no GitHub. As visualizações inicialmente apresentadas pelo `display()` no Databricks não eram reproduzidas da mesma forma no GitHub. Para garantir que os resultados das análises ficassem registrados no notebook publicado, as visualizações foram adaptadas para gráficos utilizando Matplotlib.

---

## 3. Aprendizados

O desenvolvimento do MVP permitiu consolidar conhecimentos que anteriormente estavam mais concentrados na parte conceitual da Engenharia de Dados.

Um dos principais aprendizados foi compreender que um pipeline não consiste apenas em transformar dados. É necessário pensar também em **organização, rastreabilidade, qualidade, persistência e consumo dos dados**.

A arquitetura Medallion também ficou mais clara a partir da implementação prática. A separação entre dados brutos, dados tratados e dados analíticos facilita a manutenção do pipeline e torna mais claro o papel de cada etapa.

Outro aprendizado foi a importância da qualidade dos dados como parte integrante do pipeline. Mesmo quando uma base parece simples e adequada para análise, é necessário realizar validações antes de utilizar seus resultados.

Também foi importante perceber a diferença entre desenvolver um notebook que funciona no ambiente de execução e desenvolver um projeto que pode ser compreendido e reproduzido por outra pessoa. A documentação, o catálogo de dados, a organização do GitHub e o registro das análises fazem parte do produto final.

---

## 4. O que poderia ser melhorado

Apesar de os objetivos principais terem sido atingidos, existem pontos que poderiam ser aprimorados em uma próxima versão.

O pipeline poderia ser automatizado para que sua execução não dependesse da execução manual do notebook. Também seria possível implementar mecanismos de monitoramento da qualidade dos dados, permitindo identificar automaticamente alterações na estrutura ou problemas na fonte.

Outra evolução seria separar o pipeline em diferentes notebooks ou scripts, de acordo com as etapas de processamento. Isso poderia facilitar a manutenção e a reutilização das diferentes partes do projeto em um ambiente de produção.

Também seria possível ampliar a análise incorporando outras fontes de dados, como indicadores econômicos e variáveis de mercado, permitindo análises mais completas sobre os fatores relacionados ao comportamento do preço do ouro.

Por fim, uma evolução natural seria disponibilizar os indicadores da camada Gold em um dashboard, permitindo que os resultados fossem acompanhados de forma mais interativa.

---

## 5. Considerações finais

A realização do MVP foi importante para transformar conceitos de Engenharia de Dados em um fluxo prático e completo.

O projeto passou pela ingestão dos dados, armazenamento em nuvem, modelagem em camadas, transformação, validação da qualidade, criação de tabelas analíticas e análise dos resultados.

Além do conhecimento técnico, o desenvolvimento também reforçou a importância de documentar decisões e manter a rastreabilidade do processo. O resultado final não é apenas um conjunto de tabelas ou um notebook, mas um pipeline estruturado e documentado, com código disponível publicamente e resultados que podem ser reproduzidos a partir do projeto desenvolvido.

