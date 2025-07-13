# Projeto Visão 360 E-commerce - Dashboard Power BI vs Looker Studio

## Objetivo do projeto

Este projeto tem como objetivo comparar, de forma aplicada, duas das ferramentas mais utilizadas atualmente: Power BI (Microsoft) e Looker Studio (Google).

A análise será conduzida a partir de um dataset de e-commerce do Kanggle, e a comparação considerará aspectos fundamentais como:

- Facilidade de uso e curva de aprendizado
- Performance e tempo de carregamento
- Automatização de atualizações
- Personalização visual e design de dashboards
- Integração com outras ferramentas
- Custo e escalabilidade

Para garantir que a comparação será isonoma, no projeto será utilizado os mesmos indicadores e gráficos nas duas ferramentas (Power BI e Looker). Além disso, será implementado um pipeline completo e automatizado, com as seguintes etapas:

- **Etapa 1 - Kaggle:** API para extração do database automatizada;
- **Etapa 2 - Google Colab (*Python*):** Processo de ETL (Extract, Transformation and Load);
- **Etapa 3 - BigQuery:** Armazenamento em nuvem por meio do Google BigQuery
- **Etapa 4 - Power BI e Looker:** Criação de dashboard idênticas;
Com isso, será possível entender as vantagens e limitações de cada plataforma, oferecendo uma visão clara para profissionais e empresas que desejam tomar decisões mais assertivas na escolha de ferramentas de BI. Abaixo segue o detalhamento 

## Pipeline do projeto
<img width="1409" height="676" alt="image" src="https://github.com/user-attachments/assets/adebb825-f3fd-4f9a-8276-b62408e31cd7" />

## Etapa 1 - Kaggle - Extração Automatizada do Dataset

A base para esta análise são diversos *datasets* de *e-commerce* disponível no Kaggle, intitulado olistbr/brazilian-ecommerce. Este conjunto de dados contém informações transacionais reais da Olist, uma grande empresa brasileira de *e-commerce* que conecta pequenos negócios a clientes. Este *dataset* é composto por diversas tabelas relacionadas, que permitem uma análise detalhada de diferentes aspectos do negócio.

<img width="2486" height="1496" alt="image" src="https://github.com/user-attachments/assets/d1762a29-4bfc-48e9-b873-dafab9d20557"/>

A extração dos *datasets* será realizado direto do Google Colab por meio da API do Kaggle.

## Etapa 2 - Google Colab - Processo de ETL

O arquivo `projeto_visao360_ecommerce.ipynb`, construido no Google Colab utilizando a linguagem python, anexado neste repositório está divido em 4 partes na qual foi realizado o processo de conexão com Kaggle, ETL e conexão dos dados tratados com o *Bigquery*. Segue um resumo sobre cada parte do código:

- **Parte 1 - Conexão Kaggle <> Google Colab:** Esta etapa foca em configurar o ambiente do Google Colab para acessar os dados do Kaggle. Isso envolve montar o Google Drive para persistência e acesso a credenciais (`kaggle.json`), instalar a biblioteca Kaggle, e então usar a API do Kaggle para baixar o dataset olistbr/brazilian-ecommerce para o ambiente do Colab, descompactando-o em uma pasta específica.

- **Parte 2 - Leitura e Análise Inicial das Tabelas:** Aqui, o objetivo é carregar os diversos arquivos CSV do dataset Olist (como clientes, geolocalização, pagamentos, etc.) em DataFrames do Pandas. Para cada DataFrame, são realizadas verificações iniciais, como a exibição das primeiras linhas (`.head()`), a inspeção dos tipos de dados (`.info()`) e a contagem de valores nulos (`.isnull().sum()`), garantindo a compreensão da estrutura e qualidade dos dados brutos.

- **Parte 3 - ETL dos Datasets:** Esta é a fase de transformação. Primeiramente, são identificados e filtrados order_ids comuns entre os datasets principais (itens do pedido, informações do pedido e pagamentos) para garantir a consistência dos dados. Em seguida, as colunas de data (`timestamp`) em df_info_pedido são convertidas para o tipo datetime. Por fim, é criada uma tabela calendário (`df_calendario`) com base no período das compras, adicionando colunas de ano, mês e nome do mês para facilitar análises temporais.

- **Parte 4 - Conectando os Datasets com o BigQuery:** A etapa final do ETL consiste em carregar os DataFrames processados para o Google BigQuery. O código autentica o usuário do Colab no Google Cloud e, em seguida, itera sobre um dicionário que mapeia os nomes desejados das tabelas no BigQuery para os respectivos DataFrames do Pandas. Utiliza-se a função `to_gbq` para enviar cada DataFrame para o BigQuery, substituindo a tabela se ela já existir, garantindo que os dados transformados estejam disponíveis para consumo.

## Etapa 3 - BigQuery:

A terceira etapa do pipeline centraliza o armazenamento dos dados transformados no Google BigQuery. O BigQuery é um data warehouse em nuvem altamente escalável e sem servidor do Google Cloud. Sua função aqui é atuar como o repositório principal dos dados após o processo de ETL no Colab, recebendo as tabelas limpas e preparadas. Isso permite que os dados sejam armazenados de forma eficiente e estejam prontos para análises de larga escala.

Um ponto crucial a ser destacado é que, para otimizar a performance e contornar as limitações de modelagem direta no Looker Studio, foram criadas tabelas intermediárias agregadas diretamente no BigQuery, utilizando consultas SQL. Essas tabelas pré-processam e consolidam informações, facilitando a construção final dos dashboards. As principais tabelas intermediárias criadas são:

- `tabint_revw_ag:` Agrega dados de avaliações de pedidos, calculando a média das notas e o total de avaliações por order_id.

- `tabint_pag_ag:` Consolida informações de pagamento, somando o valor total, contando a quantidade de pagamentos e listando os tipos de pagamento por order_id.

- `tabint_itped_ag:` Agrega dados de itens do pedido, somando preço e frete, contando o total de itens, e listando IDs de produtos e vendedores por order_id.

A partir dessas tabelas intermediárias e das tabelas originais carregadas do Colab (como `pedidos, clientes, produtos, vendedores e categorias`), a tabela final `bd_datasets_agregado3` é construída. Essa tabela integra todas as informações necessárias por meio de múltiplas junções (`LEFT JOINs`) baseadas nos IDs de relacionamento, consolidando um dataset único e abrangente. Esse processo de pré-agregação garante que o Looker Studio possa acessar dados já preparados e otimizados para suas visualizações, evitando a necessidade de cálculos complexos em tempo real na ferramenta de BI.

Os códigos em SQL para criação de cada tabela intermediária (`tabint`) estarão em anexo no repósitório;

## Etapa 4 - Power BI e Looker:
Esta fase é a culminação do pipeline, onde os dados preparados no BigQuery são transformados em insights visuais por meio de dashboards interativas. A criação de dashboards idênticas em Power BI e Looker Studio é fundamental para a comparação isonômica das ferramentas. Cada dashboard apresentará os mesmos indicadores e visualizações, permitindo avaliar a performance, a facilidade de uso, a personalização e a experiência geral de ambas as plataformas.

Os principais indicadores e gráficos planejados para as dashboards incluem:

**Indicadores:**

- Receita Total: Soma de pagamentos[payment_value];
- Total de Pedidos: Contagem distintos itens_pedido[order_id];
- Ticket Médio: Receita Total / Total de Pedidos;
- Pedidos em atraso: Contagem de order_id que pedidos[order_delivered_customer_date] > pedidos[order_estimated_delivery_date]
- Avaliação dos Clientes: média de reviews[review_score];

Além de comparação vs MA (Mês Anterior) vs AA (Ano Anterior) de cada indicador acima;

**Parâmetros:**
De acordo com o parametros definido entre (Receita, Pedidos e Ticket Médio) os gráficos vão se alterando a análise definida;

**Gráficos:**
- Série temporal: Gráfico de linhas mostrando a evolução mensal/anual de acordo com o parâmetro definido;
- Análie por Tipo de Pagamento: Gráfico de rosca mostrando a distribuição do parâmetro definido em relação ao tipo de pagamento;
- Análise por Status da Venda: Gráfico de rosca mostrando a distribuição do parâmetro definido em relação ao status da venda (desconsiderando os pedidos que já foi entregue "delivered");
- Distribuição Geográfica: Mapa mostrando distribuição do parâmetro definido por cidade e UF;
- Detalhamento dos indicadores por categoria de produto: Tabela com detalhamento dos 5 indicadores de acordo com a categoria dos produtos. 

LINK DASHBOARD POWER BI -
LINK DASHBOARD LOOKER - https://lookerstudio.google.com/reporting/8a85399b-7788-4bd3-959e-3e93ea45def2
