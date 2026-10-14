# Granite & Solid Surface Sales Analytics

## Importação de base de dados CSV e criação do Executive Dashboard

![Power BI](https://img.shields.io/badge/Tool-Power%20BI-yellow)
![Portfolio](https://img.shields.io/badge/Project-Portfolio-blue)
![Period](https://img.shields.io/badge/Period-2024-lightgrey)

## 1. Objetivo

O Projeto de Extensão 4 tem como objetivo transformar uma base de vendas em CSV em um dashboard
interativo no Power BI para acompanhar faturamento, despesas, lucro,
margem, volume vendido, materiais e distribuição geográfica de uma
empresa de Granite & Solid Surface durante o ano de 2024.

Este relatório documenta a preparação dos dados e a criação do
**Executive Dashboard**. 

## 2. Base de dados

**Arquivo de origem:** `excel_base(1).csv`

  Indicador                                      Valor observado
  ------------------------------------- ------------------------
  Registros                                                1.179
  Período                                 02/01/2024--20/12/2024
  Área vendida                                    53.014,69 sqft
  Faturamento                                  US\$ 3.326.379,48
  Despesas                                     US\$ 2.162.146,00
  Lucro                                        US\$ 1.164.233,48
  Margem calculada                           Aproximadamente 35%
  Materiais distintos (`MATERIAL_ID`)                        112
  Clientes distintos                                       1.028
  Estados                                            MD, VA e DC

> Valores de referência da análise inicial. Valide os totais no Power BI
> depois da importação, pois tipos de dados, localidade e transformações
> podem afetar os resultados.

### Dicionário de dados

  Coluna             Descrição
  ------------------ ----------------------------------------------
  `MATERIAL_ID`      Código do material
  `MATERIAL_NAME`    Nome ou marca do material
  `MATERIAL_TYPE`    Categoria: Granite ou Solid Surface
  `MATERIAL_COLOR`   Cor ou padrão
  `PRICE_SQFT`       Preço por pé quadrado
  `SELL_DATE`        Data da venda
  `SQFT_SOLD`        Quantidade vendida em sqft
  `SALES_TOTAL`      Valor total da venda
  `EXPENSES`         Despesas/custos registrados
  `PROFIT`           Lucro registrado
  `CUSTOMER`         Cliente
  `CITY_NAME`        Cidade
  `COUNTY_NAME`      Condado
  `STATE`            Estado
  `TYPE`             Tipo de cliente, como Customer ou Contractor
  `TRIMESTRE`        Trimestre informado na origem

**Importante:** `MATERIAL_ID` identifica o material, não uma venda
individual. O mesmo código pode aparecer em várias transações; não
remova registros apenas porque o ID se repete.

## 3. Passos realizados na Importação do CSV

1.  Aberto o Power BI Desktop.
2.  Selecionado **Home → Get Data → Text/CSV**.
3.  Selecionado arquivo a ser imnportado `excel_base(1).csv`.
4.  Realizada a conferência prévia.
5.  Clicado em **Transform Data** para abrir o Power Query.
6.  Renomeada a consulta para `Sales`.

## 4. Prepararação dos dados no Power Query

Definido `SELL_DATE` como **Date**.

Definido como **Decimal Number**: - `PRICE_SQFT` - `SQFT_SOLD` -
`SALES_TOTAL` - `EXPENSES` - `PROFIT`

Definido como **Text**: - `MATERIAL_ID`, `MATERIAL_NAME`,
`MATERIAL_TYPE`, `MATERIAL_COLOR` - `CUSTOMER`, `CITY_NAME`,
`COUNTY_NAME`, `STATE`, `TYPE`, `TRIMESTRE`

As datas e/ou números interpretados incorretamente tiveram a
localidade adequada corrigida ao formato do arquivo. Foi conferido: nulos, erros de
conversão e separadores decimais. Depois de aplicadas as correções o arquivo foi salvo.

## 5. Criaç!ao da tabela calendário

Selecionado **Modeling → New Table** e inserir:

``` dax
Calendar =
CALENDAR(
    MIN(Sales[SELL_DATE]),
    MAX(Sales[SELL_DATE])
)
```

Foram criadas as colunas:

``` dax
Year =
YEAR(Calendar[Date])
```

``` dax
Month Number =
MONTH(Calendar[Date])
```

``` dax
Month =
FORMAT(Calendar[Date], "MMMM")
```

``` dax
Quarter =
"Q" & FORMAT(Calendar[Date], "Q")
```

``` dax
Year Quarter =
FORMAT(Calendar[Date], "YYYY") &
"-Q" &
FORMAT(Calendar[Date], "Q")
```

``` dax
Month Year =
FORMAT(Calendar[Date], "MMM YYYY")
```

E pra ordenar os meses cronologicamente, foi criado:

``` dax
Month Year Sort =
YEAR(Calendar[Date]) * 100 +
MONTH(Calendar[Date])
```

Selecionado `Calendar[Month Year]` e usado **Column tools → Sort by column →
Month Year Sort**.

### Relacionamentos

Na visão **Model**, foi relacionado `Calendar[Date]` com `Sales[SELL_DATE]`.
Configure \*\*One-to-many (1:\*)\*\*, com `Calendar` no lado 1 e filtro
em direção única de `Calendar` para `Sales`. Marque `Calendar` como
tabela de datas.

## 6. Criação das medidas DAX

Criada cada medida em **Modeling → New Measure**.

### Indicadores principais

``` dax
Total Sales =
SUM(Sales[SALES_TOTAL])
```

``` dax
Total Expenses =
SUM(Sales[EXPENSES])
```

``` dax
Total Profit =
SUM(Sales[PROFIT])
```

``` dax
Total Sqft =
SUM(Sales[SQFT_SOLD])
```

``` dax
Number of Sales =
COUNTROWS(Sales)
```

> `Number of Sales` conta linhas. Só equivale ao número de transações se
> cada linha representar uma transação.

``` dax
Customers =
DISTINCTCOUNT(Sales[CUSTOMER])
```

``` dax
Materials =
DISTINCTCOUNT(Sales[MATERIAL_ID])
```

### Médias e margem

``` dax
Average Sale =
DIVIDE([Total Sales], [Number of Sales])
```

``` dax
Average Price Sqft =
DIVIDE([Total Sales], [Total Sqft])
```

``` dax
Profit Margin =
DIVIDE([Total Profit], [Total Sales])
```

``` dax
Sales per Customer =
DIVIDE([Total Sales], [Customers])
```

Formatadas as medidas financeiras como moeda e `Profit Margin` como
percentual. `DIVIDE` lida com denominadores iguais a zero de forma
segura.

## 7. Construção do Executive Dashboard

Criada uma página chamada `Executive Dashboard`.

### 7.1 Título

**SALES & PROFITABILITY DASHBOARD**\
Subtítulo: **Granite & Solid Surface Sales --- 2024**

### 7.2 KPIs

Criação dos visuais **Card**:

  KPI             Medida
  --------------- -------------------
  Total Sales     `[Total Sales]`
  Total Profit    `[Total Profit]`
  Profit Margin   `[Profit Margin]`
  Total Sqft      `[Total Sqft]`
  Customers       `[Customers]`

### 7.3 Tendência de vendas

Visual: **Line Chart** - X-axis: `Calendar[Year Quarter]` - Y-axis:
`[Total Sales]`

### 7.4 Vendas por estado

Visual: **Clustered Column Chart** - Axis: `Sales[STATE]` - Values:
`[Total Sales]`

### 7.5 Vendas versus lucro

Visual: **Line and Clustered Column Chart** - Axis:
`Calendar[Year Quarter]` - Column values: `[Total Sales]` - Line values:
`[Total Profit]`

### 7.6 Filtros

Adicionar slicers para: - `Calendar[Year]` - `Sales[STATE]` -
`Sales[MATERIAL_TYPE]` - `Sales[TYPE]`

Testado se os filtros atualizam os visuais corretamente. Quando necessário,
usado **Format → Edit interactions**.

## 8. Criação do Layout do Dashboard

1.  Título e período analisado.
2.  Linha superior de KPIs.
3.  Tendência de vendas por trimestre.
4.  Comparação de vendas por estado.
5.  Faturamento versus lucro.
6.  Slicers organizados numa área consistente.

Usado fundo claro, títulos legíveis e poucas cores de destaque.

## 9. Validação e critérios de conclusão

-   [ ] CSV importado e número de linhas conferido.
-   [ ] Tipos de dados revisados.
-   [ ] Tabela `Calendar` criada.
-   [ ] Relacionamento `Calendar` → `Sales` funcionando.
-   [ ] Medidas DAX criadas e formatadas.
-   [ ] Totais de vendas, despesas, lucro e sqft validados.
-   [ ] Slicers testados.
-   [ ] Arquivo salvo como `Granite_Sales_Dashboard.pbix`.

## 10. Resultado esperado

Ao concluir este relatório, o projeto tem os dados preparados, uma
tabela calendário, medidas DAX reutilizáveis e um **Executive
Dashboard** para resumir o desempenho de 2024.

**Próxima fase a ser construida no Projeto de Extensão 5:**  documentar Sales Analysis, Material Analysis, Geographic Analysis,
Customer Analysis e análise de rentabilidade.
