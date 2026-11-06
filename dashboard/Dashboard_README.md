# Power BI Dashboard (Member 4: Khadijat Ozohu Yusuf)

This section explains how the Group B dashboard was built, what each page shows, and how to reproduce it. The dashboard combines the cleaned data (Mugisha), the SQL sales analysis (Muluka) and the forecasts and restock recommendations (Molemi) into one interactive report that answers the project question: *how much of each product will a small shop sell in the coming weeks, and what should it buy more of?*

### 1. Where things are stored

| Item | Location |
|---|---|
| Power BI file (.pbix) | Google Drive `dashboard` folder and GitHub `dashboard` folder |
| Input CSV files | GitHub `outputs` folder and Google Drive `outputs` folder (`top_10_products.csv` is in Drive `data/processed`) |
| Slides | Google Drive `slides` folder |

Large raw data files are not committed to GitHub. No passwords, tokens or personal details are included in the report.

### 2. Data sources

| File | Owner | Rows | Columns | Used on |
|---|---|---|---|---|
| `top_10_products.csv` | Mugisha | 10 | StockCode, Description, TotalQuantity, TotalRevenue | Product table; Product Performance |
| `monthly_sales_trend.csv` | Muluka | 13 | Month, Total_Revenue | Business Overview |
| `sales_by_country.csv` | Muluka | 5 | Country, Total_Revenue | Business Overview |
| `top_revenue_products.csv` | Muluka | 10 | StockCode, Description, Total_Revenue | Product Performance |
| `slow_moving_products.csv` | Muluka | 10 | StockCode, Description, Total_Sold | Product Performance |
| `top_10_sql_check.csv` | Muluka | 10 | StockCode, Description, Total_Sold | Loaded for checking; not shown on a page |
| `forecasts.csv` | Molemi | 120 (10 products x 12 weeks) | week_start, StockCode, actual, baseline_forecast, model_forecast | Demand Forecasting; Model Evaluation |
| `restock.csv` | Molemi | 10 | StockCode, Description, weekly_forecast, usual_weekly_sales, reorder_point, assumed_stock, reorder, order_at_least | Stock Recommendations |

### 3. Build steps

**Step 1. Load.** Each CSV was loaded with Home > Get Data > Text/CSV > Transform Data.

**Step 2. Set data types in Power Query.**
- `StockCode` set to **Text** in every table except `slow_moving_products` (codes such as `85099B` contain letters, and joins fail if the types differ). `slow_moving_products` keeps it as a whole number: all its codes are numbers, and the table has no relationships.
- In `monthly_sales_trend (1)`, `Month` was set to Date. The monthly chart uses its Year and Month levels, so the two Decembers (2010 and 2011) stay separate.
- In `forecasts`, `week_start` was set to Date, `actual` to Whole Number, and the two forecast columns to Decimal Number; blank actuals were kept as null.
- The tables in the file are named `monthly_sales_trend (1)`, `sales_by_country (1)`, `top_10_products (2)`, `top_revenue_products`, `slow_moving_products`, `top_10_sql_check`, `forecasts` and `restock`.
- The file's data sources point to a local Downloads folder. On another computer the saved data still displays, but to refresh it, point each source to your own copy of the CSV files (Transform Data > Data source settings).

**Step 3. Calculated columns.**
```
forecasts[Period] = IF(ISBLANK(forecasts[actual]), "Forecast", "Actual")

'top_10_products (2)'[List Overlap] =
IF(ISBLANK(LOOKUPVALUE(top_revenue_products[StockCode], top_revenue_products[StockCode], 'top_10_products (2)'[StockCode])),
   "In this list only", "In both top 10 lists")
```
A matching `List Overlap` column was added to `top_revenue_products`. These two `List Overlap` columns are not currently shown on a page.

**Step 4. Data model.**

| From | To | Cardinality | Filter direction |
|---|---|---|---|
| `top_10_products (2)[StockCode]` | `forecasts[StockCode]` | One to many | Single |
| `top_10_products (2)[StockCode]` | `restock[StockCode]` | One to one | Both |

`top_10_products (2)` is the product dimension. `monthly_sales_trend (1)`, `sales_by_country (1)`, `top_revenue_products`, `slow_moving_products` and `top_10_sql_check` are intentionally left **unrelated**: they are SQL summaries at their own level of detail, and the revenue and slow-moving lists contain products outside the top 10, so relationships would only create blank rows. No separate date table was built; the monthly chart uses Power BI's automatic date levels.

**Step 5. Measures** (held in the `Measures (2)` table).
```
Total Revenue        = SUM('monthly_sales_trend (1)'[Total_Revenue])
Peak Month Revenue   = MAX('monthly_sales_trend (1)'[Total_Revenue])
UK Share of Top 5    = DIVIDE(CALCULATE(SUM('sales_by_country (1)'[Total_Revenue]),
                         'sales_by_country (1)'[Country] = "United Kingdom"),
                         SUM('sales_by_country (1)'[Total_Revenue]))
Top 10 Units         = SUM('top_10_products (2)'[TotalQuantity])
Top 10 Revenue       = SUM('top_10_products (2)'[TotalRevenue])
Top 10 Share of Revenue = DIVIDE([Top 10 Revenue], [Total Revenue])

Actual Units         = SUM(forecasts[actual])
Baseline Forecast    = SUM(forecasts[baseline_forecast])
ML Forecast          = SUM(forecasts[model_forecast])
Baseline Next 3 Weeks = CALCULATE([Baseline Forecast], forecasts[Period] = "Forecast",
                         forecasts[week_start] < DATE(2011,12,26))
ML Next 3 Weeks      = CALCULATE([ML Forecast], forecasts[Period] = "Forecast",
                         forecasts[week_start] < DATE(2011,12,26))

MAE Baseline  = AVERAGEX(FILTER(forecasts, NOT ISBLANK(forecasts[actual])),
                  ABS(forecasts[actual] - forecasts[baseline_forecast]))
MAE ML        = AVERAGEX(FILTER(forecasts, NOT ISBLANK(forecasts[actual])),
                  ABS(forecasts[actual] - forecasts[model_forecast]))
RMSE Baseline = SQRT(AVERAGEX(FILTER(forecasts, NOT ISBLANK(forecasts[actual])),
                  (forecasts[actual] - forecasts[baseline_forecast]) ^ 2))
RMSE ML       = SQRT(AVERAGEX(FILTER(forecasts, NOT ISBLANK(forecasts[actual])),
                  (forecasts[actual] - forecasts[model_forecast]) ^ 2))
Better Model  = IF([MAE ML] < [MAE Baseline], "ML model", "Baseline")

Products To Reorder = CALCULATE(COUNTROWS(restock), restock[reorder] = "Yes")
Units To Order      = SUM(restock[order_at_least])

Avg Price per Unit   = DIVIDE(SUM('top_10_products (2)'[TotalRevenue]), SUM('top_10_products (2)'[TotalQuantity]))
SQL Check Difference = SUM('top_10_products (2)'[TotalQuantity]) - SUM(top_10_sql_check[Total_Sold])
```
The last two measures are not currently shown on a page.

### 4. Dashboard pages

| Page | Purpose | Main visuals |
|---|---|---|
| **Business Overview** | Size of the business, when sales peak and where they come from | Cards (Total Revenue, Peak Month Revenue, UK Share of Top 5); monthly revenue column chart; top 5 countries bar chart |
| **Product Performance** | Which products matter by volume and by value, and which barely sell | Cards (Top 10 Units, Revenue and Share of Revenue); top 10 by units; top 10 by revenue; scatter of units vs revenue; table of products that sold one unit |
| **Demand Forecasting** | What the two methods expect to sell over the next three weeks | Cards (Baseline and ML next 3 weeks); line chart of actual vs baseline vs ML; product dropdown; product table |
| **Model Evaluation** | How accurate the two forecasting approaches are | Cards (MAE and RMSE for each model); MAE by product (clustered bar); table with Better Model |
| **Stock Recommendations** | What the shop should buy | Cards (Products To Reorder, Units To Order); restock table with colour rules; stock vs reorder point chart; units to order chart; assumptions box |

### 5. Interactivity

- **Product dropdown** on the Demand Forecasting page, which filters that page's cards and line chart.
- **Cross-highlighting:** Power BI's standard behaviour, where clicking an item in one visual highlights or filters the others on the same page.
- **Tooltips:** Power BI's standard hover tooltips on charts.
- A **page-level filter** hides the week of 26 Dec 2011 on the Demand Forecasting page (see limitations).

### 6. Design standards

- 16:9 canvas, with the same navy header band (`#1F3A5F`) on every page.
- Colours on the forecast and evaluation charts: Actual = light blue, Baseline = orange, ML model = navy.
- Restock table: Yes = red, No = blue.
- Many chart titles state the finding rather than the field name, such as "Revenue peaked in November 2011 ahead of Christmas".

### 7. Key findings shown in the dashboard

- Total revenue across the 13 months is about **£9.71M**, peaking at **£1.43M in November 2011**. December 2011 is a partial month, because the data stops on 9 December.
- The UK accounts for about **90%** of revenue among the five countries listed.
- The top 10 products by quantity sold **369,877 units** but earned about **£430K (4.4% of revenue)**. Only five products appear in both the top-10 volume and top-10 revenue lists.
- The 10 slowest products listed sold only one unit each. They are tied, so they are examples, and other products may also have sold only one unit.
- Over the 8 weeks with actual sales, the **baseline had lower error (MAE 593.9, RMSE 1,028.2) than the ML model (MAE 708.1, RMSE 1,201.5)**. The ML model was better on three products: 22492, 84077 and 85099B.
- Six of ten products fall below their reorder point, with at least **17,395 units** to order. Rabbit Night Light (8,125) and Small Popcorn Holder (6,581) make up about 85% of that.

### 8. Assumptions and limitations

- The dataset has no stock data, so `assumed_stock` is an **assumption**, shown on the Stock Recommendations page.
- The restock figures follow the baseline forecast: the weekly forecast equals the baseline average for 5 to 19 December, the reorder point is three weeks of forecast demand, and the order quantity is the reorder point minus assumed stock.
- The forecast pages show **three forecast weeks** (5, 12 and 19 December 2011). The week of 26 December is set to zero for every product because the shop is assumed to be closed, and it is hidden from the Demand Forecasting page.
- The ML forecast after the red line on the chart comes from the model retrained on all weeks, so it has not been tested. The MAE and RMSE scores come from the 8 hidden weeks.
- Forecasts cover only the top 10 products by quantity, not all ~4,000 products or the revenue leaders.
- The country file lists only five countries, so percentages are shares of those five.
- Rabbit Night Light sales jumped from near zero to thousands of units a week in late October; its forecast is about six times its usual weekly sales and should be read with that in mind.

### 9. How to reproduce

1. Download the eight CSV files from the shared Drive folder into one local folder.
2. Open Power BI Desktop and load each file, following Steps 1 and 2 above.
3. Add the calculated columns, create the two relationships and add the measures (Steps 3 to 5).
4. Build each page as described in section 4, then set up the interactivity in section 5.
5. Compare the numbers with the key findings in section 7.
6. Save the .pbix to the Drive `dashboard` folder and export a PDF copy.
