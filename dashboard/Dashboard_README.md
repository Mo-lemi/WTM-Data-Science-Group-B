Power BI Dashboard (Member 4: Khadijat Ozohu Yusuf)

This section explains how the Group B dashboard was built, what each page shows, and how to reproduce it. The dashboard combines the cleaned data (Mugisha), the SQL sales analysis (Muluka) and the forecasts and restock recommendations (Molemi) into one interactive report that answers the project question: *how much of each product will a small shop sell in the coming weeks, and what should it buy more of?*

1. Where things are stored

| Item | Location |
|---|---|
| Power BI file (.pbix) | Google Drive, `dashboard` folder |
| Input CSV files | Google Drive, `data/processed` and `outputs` folders |
| Slides and backup demo video | Google Drive, `slides` folder |

Large raw data files and the .pbix are not committed to GitHub. No passwords, tokens or personal details are included in the report.

### 2. Data sources

| File | Owner | Rows | Columns | Used on |
|---|---|---|---|---|
| `top_10_products.csv` | Mugisha | 10 | StockCode, Description, TotalQuantity, TotalRevenue | Product table; Product Performance |
| `monthly_sales_trend.csv` | Muluka | 13 | Month, Total_Revenue | Business Overview |
| `sales_by_country.csv` | Muluka | 5 | Country, Total_Revenue | Business Overview |
| `top_revenue_products.csv` | Muluka | 10 | StockCode, Description, Total_Revenue | Product Performance |
| `slow_moving_products.csv` | Muluka | 10 | StockCode, Description, Total_Sold | Product Performance |
| `top_10_sql_check.csv` | Muluka | 10 | StockCode, Description, Total_Sold | Validation |
| `forecasts.csv` | Molemi | 120 (10 products x 12 weeks) | week_start, StockCode, actual, baseline_forecast, model_forecast | Demand Forecasting; Model Evaluation |
| `restock.csv` | Molemi | 10 | StockCode, Description, weekly_forecast, usual_weekly_sales, reorder_point, assumed_stock, reorder, order_at_least | Restock Recommendations |

 3. Build steps

**Step 1. Load.** Each CSV was loaded with Home > Get Data > Text/CSV > Transform Data.

**Step 2. Clean in Power Query.**
- `StockCode` set to **Text** in every table (codes such as `85099B` contain letters, and joins fail if the types differ).
- Trailing spaces removed from every `Description` column (Transform > Format > Trim and Clean).
- In `Monthly_Sales`, a `Month Date` column was added with `Date.FromText([Month] & "-01")` so months sort correctly and the two Decembers (2010 and 2011) are not merged.
- In `Forecasts`, `week_start` was set to Date and the three numeric columns to Decimal Number; blank actuals were kept as null.
- Queries were renamed: `Monthly_Sales`, `Sales_By_Country`, `Top10_Products`, `Top_Revenue_Products`, `Slow_Moving`, `SQL_Check`, `Forecasts`, `Restock`.

**Step 3. Calculated columns.**
```
Forecasts[Period] = IF(ISBLANK(Forecasts[actual]), "Forecast", "Actual")

Top10_Products[List Overlap] =
IF(ISBLANK(LOOKUPVALUE(Top_Revenue_Products[StockCode], Top_Revenue_Products[StockCode], Top10_Products[StockCode])),
   "In this list only", "In both top 10 lists")
```
A matching `List Overlap` column was added to `Top_Revenue_Products`.

**Step 4. Data model.**

| From | To | Cardinality | Filter direction |
|---|---|---|---|
| `Top10_Products[StockCode]` | `Forecasts[StockCode]` | One to many | Single |
| `Top10_Products[StockCode]` | `Restock[StockCode]` | One to one | Both |

`Top10_Products` is the product dimension. `Monthly_Sales`, `Sales_By_Country`, `Top_Revenue_Products`, `Slow_Moving` and `SQL_Check` are intentionally left **unrelated**: they are SQL summaries at their own level of detail, and the revenue and slow-moving lists contain products outside the top 10, so relationships would only create blank rows. No separate date table is used because only `Forecasts` holds weekly dates.

**Step 5. Measures** (held in a `_Measures` table).
```
Total Revenue        = SUM(Monthly_Sales[Total_Revenue])
Peak Month Revenue   = MAX(Monthly_Sales[Total_Revenue])
UK Share of Top 5    = DIVIDE(CALCULATE(SUM(Sales_By_Country[Total_Revenue]),
                         Sales_By_Country[Country] = "United Kingdom"),
                         SUM(Sales_By_Country[Total_Revenue]))
Avg Price per Unit   = DIVIDE(SUM(Top10_Products[TotalRevenue]), SUM(Top10_Products[TotalQuantity]))
Top 10 Units         = SUM(Top10_Products[TotalQuantity])
Top 10 Revenue       = SUM(Top10_Products[TotalRevenue])
Top 10 Share of Revenue = DIVIDE([Top 10 Revenue], [Total Revenue])

Actual Units         = SUM(Forecasts[actual])
Baseline Forecast    = SUM(Forecasts[baseline_forecast])
ML Forecast          = SUM(Forecasts[model_forecast])
Baseline Next 3 Weeks = CALCULATE([Baseline Forecast], Forecasts[Period] = "Forecast",
                         Forecasts[week_start] < DATE(2011,12,26))
ML Next 3 Weeks      = CALCULATE([ML Forecast], Forecasts[Period] = "Forecast",
                         Forecasts[week_start] < DATE(2011,12,26))
Forecast Difference  = [ML Next 3 Weeks] - [Baseline Next 3 Weeks]

MAE Baseline  = AVERAGEX(FILTER(Forecasts, NOT ISBLANK(Forecasts[actual])),
                  ABS(Forecasts[actual] - Forecasts[baseline_forecast]))
MAE ML        = AVERAGEX(FILTER(Forecasts, NOT ISBLANK(Forecasts[actual])),
                  ABS(Forecasts[actual] - Forecasts[model_forecast]))
RMSE Baseline = SQRT(AVERAGEX(FILTER(Forecasts, NOT ISBLANK(Forecasts[actual])),
                  (Forecasts[actual] - Forecasts[baseline_forecast]) ^ 2))
RMSE ML       = SQRT(AVERAGEX(FILTER(Forecasts, NOT ISBLANK(Forecasts[actual])),
                  (Forecasts[actual] - Forecasts[model_forecast]) ^ 2))
Better Model  = IF([MAE ML] < [MAE Baseline], "ML model", "Baseline")

Products To Reorder = CALCULATE(COUNTROWS(Restock), Restock[reorder] = "Yes")
Units To Order      = SUM(Restock[order_at_least])
SQL Check Difference = SUM(Top10_Products[TotalQuantity]) - SUM(SQL_Check[Total_Sold])
```

4. Dashboard pages

| Page | Purpose | Main visuals |
|---|---|---|
| **Business Overview** | Size and seasonality of the business | Cards (Total Revenue, Peak Month Revenue, UK Share of Top 5); monthly revenue column chart; top 5 countries bar chart |
| **Product Performance** | Which products matter by volume and by value, and which barely sell | Top 10 by units; top 10 by revenue; scatter of units vs revenue; slow-movers table |
| **Demand Forecasting** | What the models expect to sell over the next three weeks | Cards (Baseline and ML next 3 weeks); line chart of actual vs baseline vs ML; product slicer; product table |
| **Model Evaluation** | How accurate the two forecasting approaches are | Cards (MAE and RMSE for each model); MAE by product (clustered bar); table with Better Model |
| **Restock Recommendations** | What the shop should buy | Cards (Products To Reorder, Units To Order); restock table with colour rules; stock vs reorder point chart; units to order chart; assumptions box |

5. Interactivity

- **Product slicer** on the Demand Forecasting page (dropdown with Select all), synced to the Model Evaluation and Restock pages.
- **Cross-highlighting** between the Units bar chart and the scatter chart; clicking a row in the forecast table filters the line chart.
- **Drill-through** from Product Performance to the Demand Forecasting and Restock pages using `Description`.
- **Dynamic chart title** that changes with the product selection.
- **Tooltips** with units, revenue, average price and forecast difference.
- **Page navigator** on every page and a **Reset filters** bookmark button.
- A **page-level filter** hides the week of 26 Dec 2011 on the forecast pages (see limitations).

6. Design standards

- 16:9 canvas, one font (Segoe UI), the same navy header band on every page.
- Colour coding used consistently: Actual = grey `#7F7F7F`, Baseline = orange `#ED7D31`, ML model = blue `#2E75B6`, Reorder = red `#C0392B`, No reorder = green `#2E8B57`, header = navy `#1F3A5F`.
- Chart titles state the finding rather than the field name.
- Cards and charts share white backgrounds, light borders and rounded corners; the page background is a light blue-grey.

 7. Key findings shown in the dashboard

- Total revenue across the 13 months is about **£9.71M**. It rises into the holiday season, peaking at **£1.43M in November 2011**. December 2011 is a partial month.
- The UK accounts for about **90%** of revenue among the five countries listed.
- The top 10 products by quantity sold **369,877 units** but earned about **£430K (4.4% of revenue)**. Only five products appear in both the top-10 volume and top-10 revenue lists.
- Ten products sold only one unit.
- Over the 8 weeks with actual sales, the **baseline had lower error (MAE 594, RMSE 1,028) than the ML model (MAE 708, RMSE 1,201)**. The ML model was better on three products: 22492, 84077 and 85099B.
- Six of ten products fall below their reorder point, with at least **17,395 units** to order. Rabbit Night Light (8,125) and Small Popcorn Holder (6,581) make up 85% of that.

8. Assumptions and limitations

- The dataset has no stock data, so `assumed_stock` is an **assumption**, shown on the Restock page.
- The restock figures follow the baseline forecast: the weekly forecast equals the baseline average for 5 to 19 December, the reorder point is three weeks of forecast demand, and the order quantity is the reorder point minus assumed stock.
- Only **three forecast weeks** (5, 12 and 19 December 2011) hold real values; the week of 26 December returns zero for every product and is excluded from the forecast pages.
- Forecasts cover only the top 10 products by quantity, not all ~4,000 products or the revenue leaders.
- The country file lists only five countries, so percentages are shares of those five.
- Rabbit Night Light sales jumped from near zero to thousands of units a week in late October; its forecast is about six times its usual weekly sales and should be read with that in mind.

9. How to reproduce

1. Download the eight CSV files from the shared Drive folder into one local folder.
2. Open Power BI Desktop and load each file, following Steps 1 and 2 above.
3. Add the calculated columns, create the two relationships and the measures (Steps 3 to 5).
4. Build each page as described in section 4, then add the interactivity in section 5.
5. Compare the numbers against the validation table in section 9.
6. Save the .pbix to the Drive `dashboard` folder and export a PDF copy.
