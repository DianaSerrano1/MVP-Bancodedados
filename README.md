
# MVP – Engenharia de Dados | PUC-Rio - Diana Serano

Este projeto tem como objetivo a construção de um pipeline de dados em nuvem, utilizando conceitos e práticas de Engenharia de Dados, com foco na coleta, transformação, modelagem e análise de dados estruturados.

O trabalho foi desenvolvido no contexto da disciplina de Engenharia de Dados da Pós-graduação em Ciência de Dados da PUC-Rio e utiliza dados reais de uma empresa do setor varejista de artigos de viagem, abrangendo informações de produtos, lojas e faturamento.

A solução proposta contempla a utilização da plataforma Databricks como ambiente de processamento e armazenamento, adotando uma arquitetura em camadas (Bronze, Silver e Gold) e modelagem em esquema estrela, com o objetivo de organizar os dados de forma escalável, auditável e analiticamente eficiente.

Ao final do pipeline, são realizadas análises exploratórias e analíticas com o propósito de responder perguntas de negócio relacionadas ao desempenho comercial das lojas, ao mix de produtos e à escala de vendas, respeitando critérios de qualidade dos dados e boas práticas de engenharia.

## 2. Objetivo do Trabalho

O objetivo deste MVP é desenvolver um pipeline de dados em nuvem capaz de transformar dados operacionais brutos em informações analíticas estruturadas, permitindo a análise do desempenho de vendas no nível de loja e produto.

A partir dos dados disponibilizados, o trabalho busca responder a perguntas de negócio relacionadas à escala de vendas, composição do mix de produtos e diferenças de desempenho entre lojas, utilizando métricas de volume de peças vendidas, faturamento e participação de categorias.

As análises realizadas neste projeto são guiadas pelas seguintes perguntas:

1 Desempenho por Categoria

1. Quais categorias de produto apresentam maior contribuição para o faturamento mensal da empresa?
2. Em quais períodos (meses/estações) cada categoria performa melhor ou pior?
3. Quais categorias possuem maior potencial de crescimento além das malas, que hoje representam a maior fatia da receita?

2 Desempenho por Região

1. Identificar quais estados demonstram maior abertura para categorias que não sejam apenas malas.
2. Avaliar em quais regiões a marca é percebida de forma mais ampla, consumindo outras categorias.

3 Eficiência Comercial
1. Qual estado apresenta o melhor PA (peças por atendimento) médio?
2. Há diferenças de performance entre lojas de shopping e lojas de rua?
3. Quais categorias possuem maior ticket médio por estado/loja?


Essas perguntas guiam tanto a modelagem das dimensões quanto a estrutura da tabela fato.

Durante a fase inicial de exploração dos dados, foram consideradas perguntas adicionais relacionadas à eficiência comercial, como métricas de atendimento e ticket médio por loja. No entanto, ao longo do desenvolvimento do pipeline e da análise de qualidade dos dados, identificou-se a ausência de informações necessárias para responder a essas questões de forma consistente.

Diante disso, o escopo analítico foi refinado, priorizando perguntas plenamente suportadas pelos dados disponíveis, com foco em métricas de volume de vendas, faturamento e composição do mix de produtos. Esse processo de refinamento garantiu maior robustez às análises realizadas e alinhamento entre os objetivos do trabalho e as evidências extraídas dos dados.

## 3. Fonte e Coleta dos Dados

Os dados utilizados neste projeto são provenientes de uma única fonte interna, correspondente a uma empresa do setor varejista de artigos de viagem. Para fins deste MVP, os dados foram disponibilizados em formato CSV para uso analítico, abrangendo informações de produtos, lojas e faturamento.

Apesar de os dados terem origem em sistemas corporativos, neste trabalho eles são tratados exclusivamente como arquivos estruturados em CSV, sem a necessidade de acesso direto aos sistemas transacionais. Essa abordagem permitiu focar no desenvolvimento do pipeline de dados e nas etapas de transformação e análise, em conformidade com os objetivos acadêmicos do projeto.

Foram utilizados três conjuntos de dados principais:
- **Produtos**, contendo atributos descritivos e categóricos dos itens comercializados;
- **Lojas**, contendo informações cadastrais e de localização dos pontos de venda;
- **Faturamento**, contendo registros de vendas e devoluções, com aproximadamente 900 mil linhas.

Os dados de faturamento contemplam exclusivamente o ano de 2025, organizados por período no formato ano-mês, com registros até o início de dezembro. Em função disso, o mês de dezembro foi desconsiderado em algumas análises temporais, a fim de evitar distorções decorrentes de período incompleto.

A etapa de coleta consistiu no carregamento manual dos arquivos CSV para a plataforma Databricks, utilizando **Volumes no Unity Catalog** como camada de armazenamento. A partir desses volumes, os dados foram lidos diretamente no ambiente Spark para dar início ao pipeline de transformação.

Não foram utilizadas técnicas de web scraping, APIs ou processos automatizados de extração, uma vez que os dados já se encontravam estruturados e disponíveis para análise. Dessa forma, a etapa de coleta pode ser considerada simples, porém adequada ao escopo do MVP.

Por se tratarem de dados reais de uma empresa privada, cuidados adicionais foram adotados quanto à confidencialidade. O uso dos dados foi autorizado para fins exclusivamente acadêmicos, sem a divulgação do nome da empresa ou de informações estratégicas sensíveis, mantendo o foco nas análises agregadas e nos conceitos técnicos abordados ao longo do trabalho.


## 4. Modelagem de Dados

A modelagem deste MVP foi estruturada para suportar análises no estilo **Data Warehouse**, utilizando **Esquema Estrela (Star Schema)** na camada **Gold**. Para garantir organização, rastreabilidade e evolução do pipeline, os dados foram separados em três camadas (schemas) no Databricks:

- **Bronze**: dados brutos ingeridos dos CSVs, preservando o formato original;
- **Silver**: dados tratados e padronizados (tipos, limpeza de strings, datas e numéricos);
- **Gold**: modelo dimensional final, com dimensões e tabela fato prontas para análise.

### 4.1 Granularidade e fatos

A tabela fato foi desenhada com granularidade **mensal por loja e por produto**, alinhada ao dado disponível no conjunto de faturamento (campo `ANOMES`) e ao objetivo analítico do projeto.

A tabela **`gold.fato_vendas_mensal`** consolida as métricas:
- `QTD_VENDIDA` (soma de quantidade, incluindo devoluções quando negativas);
- `VALOR_VENDIDO` (soma de valor, incluindo devoluções quando negativas).

### 4.2 Dimensões do modelo

O esquema estrela utiliza três dimensões principais:

- **Dimensão Produto (`gold.dim_produto`)**
  - Chave natural: `COD_SKU_NATURAL` (SKU do produto)
  - Atributos descritivos e categóricos: descrição, cor, categorias (agrupamento/grupo/linha), fornecedor, coleção e preço de importação (`PRECO_IMP`).

- **Dimensão Loja (`gold.dim_loja`)**
  - Chave natural: `LOJA_NATURAL`
  - Atributos: tipo/canal (ex.: loja própria, e-commerce), nome, estado, supervisor, data de primeira compra (`PRI_COMPRA_DT`) e flag de loja ativa (`LOJA_ATIVA`).

- **Dimensão Tempo (`gold.dim_tempo`)**
  - Baseada no primeiro dia do mês (`DATA_MES`) derivado do campo `ANOMES`.
  - Atributos: `ANOMES`, `ANO`, `MES`.

### 4.3 Chaves naturais e Surrogate Keys (SK)

Apesar das chaves naturais existirem nos dados (ex.: `COD_SKU`, `LOJA` e `DATA_MES`), a camada Gold adota **Surrogate Keys (SKs)** para:

- padronizar relacionamentos entre dimensões e fato;
- simplificar joins e consumo analítico;
- permitir evolução futura (ex.: alterações em chaves naturais, mudanças de cadastro, cenários de SCD).

Assim, cada dimensão recebe uma chave substituta:
- `SK_PRODUTO`, `SK_LOJA`, `SK_TEMPO`.

A tabela fato armazena as SKs como chaves de relacionamento (FKs analíticas):
- `SK_PRODUTO`, `SK_LOJA`, `SK_TEMPO`.

Além disso, a fato também mantém as **chaves naturais** (`COD_SKU`, `LOJA`, `DATA_MES`) como colunas de contexto, facilitando auditoria e validações.

### 4.4 Esquema estrela (visão conceitual)

```mermaid
erDiagram
  DIM_PRODUTO ||--o{ FATO_VENDAS_MENSAL : "SK_PRODUTO"
  DIM_LOJA    ||--o{ FATO_VENDAS_MENSAL : "SK_LOJA"
  DIM_TEMPO   ||--o{ FATO_VENDAS_MENSAL : "SK_TEMPO"

  DIM_PRODUTO {
    INT SK_PRODUTO
    STRING COD_SKU_NATURAL
    STRING DESC_SKU
    STRING DESC_COR
    STRING DESC_AGRUPAMENTO
    STRING DESC_GRUPO
    STRING DESC_LINHA
    STRING FORNECEDOR
    STRING DESC_COLECAO
    DECIMAL PRECO_IMP
  }

  DIM_LOJA {
    INT SK_LOJA
    STRING LOJA_NATURAL
    STRING TIPO
    STRING TIPO_DESC
    STRING NOME
    STRING NOME_LIMPO
    STRING SUPERVISOR
    STRING ESTADO
    DATE PRI_COMPRA_DT
    BOOLEAN LOJA_ATIVA
  }

  DIM_TEMPO {
    INT SK_TEMPO
    DATE DATA_MES
    STRING ANOMES
    INT ANO
    INT MES
  }

  FATO_VENDAS_MENSAL {
    INT SK_TEMPO
    INT SK_LOJA
    INT SK_PRODUTO
    DATE DATA_MES
    STRING LOJA
    STRING COD_SKU
    INT QTD_VENDIDA
    DECIMAL VALOR_VENDIDO
  }



