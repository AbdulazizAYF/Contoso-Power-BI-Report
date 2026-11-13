# Contoso Retail Sales Analytics: Power BI Report

An interactive Power BI report built on the Contoso retail sample data (about 4,000 sales lines from May 2022 to March 2026). It covers sales, profit, products, customers and store performance.

---

## Key insights

| # | Insight |
|---|---|
| 1 | Total sales are **$4.37M** across **1,681 orders**, with a gross margin of **55.6%**. |
| 2 | **Computers** bring in **53.9%** of sales. Cell phones are second with 15.3%. |
| 3 | The **United States** accounts for **57.1%** of sales by customer country. |
| 4 | The **online share** of sales grew from **16%** in 2022 to **28%** in 2026 (2026 is a partial year). |
| 5 | Only **18 of 1,663 customers (1.1%)** ordered more than once, so customer retention is the biggest opportunity. |
| 6 | Sales peaked in 2024 and fell **50.9%** in 2025. **February** and **December** are the strongest months. |
| 7 | Online orders take **4.0 days** to deliver on average. |

---

## Report pages

| Page | What it answers | Main visuals |
|---|---|---|
| **Overview** | How is the business doing overall? | KPI cards with same period last year, sales vs last year line chart, sales by channel, a dynamic chart driven by field parameters |
| **Sales Trend** | How do sales change over time? | Monthly sales with a 3-month moving average, year-over-year table, month × year heatmap |
| **Products** | Which products and categories drive sales and margin? | Top 10 products, category scatter (sales vs margin), category and subcategory matrix with share of total |
| **Customers** | Who are the customers? | Customer KPIs and repeat rate, sales by age band and gender, sales by country, top 20 customers |
| **Stores** | How do physical stores and online delivery perform? | Active stores, sales per m², store size vs sales per m² scatter, store detail table, delivery days distribution |
| **Product Detail** *(drillthrough)* | What does a single product look like? | Right-click any product, then **Drill through**, to see its KPIs, monthly sales and sales by country |
| **Tooltip** *(report page tooltip)* | Quick context on hover | Sales and margin plus sales by year, shown when hovering over several charts on the Overview, Customers and Stores pages |

Every main page has **Year** and **Channel** slicers, navigation buttons, and a **Reset** button (a bookmark) that clears all selections.

---

## Data model

A star schema with one fact table and four dimension tables:

```
                 Date
                   |
Customer ----- Sales (fact) ----- Product
                   |
                 Store
```

| Table | Type | Notes |
|---|---|---|
| `Sales` | Fact | One row per order line. Calculated line amount, cost, gross amount, delivery days and channel |
| `Date` | Dimension | Trimmed to the sales date range. Marked as the date table for time intelligence |
| `Customer` | Dimension | Full name, cleaned states, and an `Order Count` column used for repeat customers |
| `Product` | Dimension | Category, subcategory, brand and a `Price Band` column |
| `Store` | Dimension | `Store Type` (Physical / Online) and `Store Status` (Active / Closed / Restructured) |

Relationships are one-to-many, from each dimension to `Sales`. `Sales[DeliveryDate]` has a second, **inactive** relationship to `Date`. Hierarchies: Calendar (Year → Quarter → Month), Product (Category → Subcategory → Product) and Geography (Continent → Country → State → City).

---

## Data preparation (Power Query)

- A `FolderPath` parameter keeps the CSV location in one place.
- The header row is promoted and every column gets the correct data type.
- Unused columns are removed and blank values are replaced (`Unknown`, `N/A`).
- New columns are added with conditional and custom columns:
  - `Channel` and `Store Type` (online store vs physical stores)
  - `Store Status` (uses the close date when the status is blank)
  - `Line Amount`, `Line Cost` and `Line Gross` (quantity × price)
  - `DeliveryDays` (delivery date − order date)
  - `Price Band` and `Month Start`
- The `Date` table is filtered to the first and last sale dates.

---

## DAX

### Measures

All measures sit in a dedicated `_Measures` table, grouped into display folders.

| Measure | DAX |
|---|---|
| Sales Amount | `SUM ( Sales[Line Amount] )` |
| Total Cost | `SUM ( Sales[Line Cost] )` |
| Gross Profit | `[Sales Amount] - [Total Cost]` |
| Margin % | `DIVIDE ( [Gross Profit], [Sales Amount] )` |
| Discount | `DIVIDE ( SUM ( Sales[Line Gross] ) - [Sales Amount], SUM ( Sales[Line Gross] ) )` |
| Quantity | `SUM ( Sales[Quantity] )` |
| Orders | `DISTINCTCOUNT ( Sales[OrderKey] )` |
| Avg Order Value | `DIVIDE ( [Sales Amount], [Orders] )` |
| Customers | `DISTINCTCOUNT ( Sales[CustomerKey] )` |
| Repeat Customers | `CALCULATE ( [Customers], Customer[Order Count] > 1 )` |
| Repeat Rate | `DIVIDE ( [Repeat Customers], [Customers] )` |
| Active Stores | `COUNTROWS ( Store )` (visual filtered to physical, active stores) |
| Sales per m2 | `DIVIDE ( [Sales Amount], SUM ( Store[SquareMeters] ) )` |
| Avg Online Delivery Days | `AVERAGE ( Sales[DeliveryDays] )` (visual filtered to online orders) |

### Calculation group: `Time Calc`

One set of time-intelligence logic that works with **any** measure:

| Item | Logic |
|---|---|
| Actual | `SELECTEDMEASURE ()` |
| Last year | `CALCULATE ( SELECTEDMEASURE (), SAMEPERIODLASTYEAR ( 'Date'[Date] ) )` |
| Change | Actual − Last year |
| Change % | (Actual − Last year) ÷ Last year |

### Field parameters

- **Metric**: switches the dynamic chart between Sales, Gross Profit, Quantity and Orders.
- **Breakdown**: switches its axis between Category, Brand, Store Country, Customer Country, Age Band and Channel.

### Calculated columns

- `Sales[Customer Age]` and `Sales[Age Band]`: the customer's age at the time of the order, grouped into 10-year bands.
- `Customer[Order Count]`: the number of orders per customer.

### Visual calculations

- **3-month moving average** on the Sales Trend page:
  `IF ( NOT ISBLANK ( PREVIOUS ( [Sales Amount], 2 ) ), MOVINGAVERAGE ( [Sales Amount], 3 ) )`
- **Share of total** in the category matrix:
  `DIVIDE ( [Sales Amount], COLLAPSEALL ( [Sales Amount], ROWS ) )`

---

## Features used

`Star schema` · `Power Query` · `Parameters` · `DAX measures` · `Calculated columns` · `Calculation group` · `Field parameters` · `Visual calculations` · `Drillthrough` · `Report page tooltip` · `Bookmarks` · `Page navigation buttons` · `Conditional formatting` · `Hierarchies` · `Visual-level filters`

---

## How to open

1. Install [Power BI Desktop](https://www.microsoft.com/power-bi/desktop) (free, Windows).
2. Download `Contoso_Simple.pbix` from this repository.
3. Open it in Power BI Desktop. The data is already inside the file, so all pages work right away.

---

## Data source

Contoso sample data (CSV, 10K version) generated by the [SQLBI Contoso Data Generator](https://github.com/sql-bi/Contoso-Data-Generator-V2). This is fictional data used for learning and demonstration.

---

## Author

**Abdulaziz Alfadhli**, Management Information Systems student at King Saud University
Microsoft Certified: Power BI Data Analyst Associate (PL-300)
[LinkedIn](https://www.linkedin.com/in/abdulaziz-alfadhli-296113372/)
