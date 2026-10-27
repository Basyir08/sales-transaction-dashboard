# RetailPulse — Power BI Sales & Discount Performance Dashboard

An end-to-end Business Intelligence project built with Microsoft Power BI on an
Indian retail transaction dataset of 55,000 orders (Sep 2019 – Sep 2024). The
project covers data cleaning in Python, star-schema data modeling, 18 DAX
measures, and an executive dashboard delivering insights on revenue, discount
behaviour, regional performance, and customer segments.

---

## Dataset Overview

| Attribute | Value |
|---|---|
| Transactions | 55,000 |
| Unique customers | 29,071 |
| Period | 19 Sep 2019 – 18 Sep 2024 |
| Gross revenue | ₹165.71M |
| Net revenue | ₹158.24M |
| Total discount | ₹7.47M (4.5% of gross) |
| Average order value | ₹2,877 |
| Product categories | 9 |
| Cities | 14 |
| Payment methods | 8 (grouped into 5) |
| Customer segments | 3 genders × 5 age groups |

## Project Scope

1. **Data cleaning (Python / pandas)** — resolved mixed date formats
   (`dd/MM/yyyy HH:mm:ss` vs `d/M/yyyy H:mm`), standardised to ISO, grouped
   8 payment methods into 5 business categories, added sort-order columns.
2. **Data modeling (Power BI)** — star schema with a dedicated `DimDate`
   calendar table marked as date table, plus optional dimension tables for
   category, location, payment method and age group.
3. **DAX measures** — 18 measures covering core metrics, time intelligence
   (previous-period comparison, YTD, MoM, 3-month rolling average), share and
   ranking, discount analysis, and data-quality checks.
4. **Dashboard** — 5 KPI cards, trend analysis, category and regional
   breakdowns, payment-method composition, and dynamic management highlights.

## Key Findings

- Credit Card dominates payments at 45.4% of net sales, not UPI (14.6%) —
  contrary to the common assumption about Indian retail.
- Discount penetration sits at 4.5% of gross revenue, with Electronics
  absorbing the largest share.
- 613 transactions (1.1%) carry a discount equal to or greater than the gross
  amount, flagged via `flag_diskon_invalid` for data-quality monitoring.

  ## DAX Measures (18)

| Measure | Formula | Purpose |
|---|---|---|
| Gross Sales | `SUM(Sales[Gross Amount])` | Total transaction value before discount |
| Net Sales | `SUM(Sales[Net Amount])` | Revenue actually received after discount |
| Total Discount | `SUM(Sales[Discount Amount])` | Total discount given across transactions |
| Transactions | `DISTINCTCOUNT(Sales[TID])` | Number of unique transactions |
| Customers | `DISTINCTCOUNT(Sales[CID])` | Number of unique customers |
| AOV | `DIVIDE([Net Sales], [Transactions])` | Average net value per transaction |
| Net Sales PP | `CALCULATE([Net Sales], DATESBETWEEN(DimDate[Date], MIN(DimDate[Date]) - (DATEDIFF(MIN(DimDate[Date]), MAX(DimDate[Date]), DAY) + 1), MIN(DimDate[Date]) - 1))` | Net sales in the equally-long period immediately before the selected range |
| Net Sales PP % | `DIVIDE([Net Sales] - [Net Sales PP], [Net Sales PP])` | Growth vs previous period, drives the ▲/▼ indicator on KPI cards |
| % of Total Net Sales | `DIVIDE([Net Sales], CALCULATE([Net Sales], ALLSELECTED(Sales[Product Category])))` | Each category's share of total net sales |
| Category Rank | `RANKX(ALLSELECTED(Sales[Product Category]), [Net Sales], , DESC, DENSE)` | Category ranking by net sales, used for Top N filtering |
| Discount Rate % | `DIVIDE([Total Discount], [Gross Sales])` | Discount as a proportion of gross sales |
| Trx with Discount | `CALCULATE([Transactions], Sales[Discount Availed] = "Yes")` | Number of transactions that used a discount |
| Discount Usage % | `DIVIDE([Trx with Discount], [Transactions])` | Share of transactions involving a discount |
| Avg Discount per Trx | `DIVIDE([Total Discount], [Trx with Discount])` | Average discount value per discounted transaction |
| Net Sales YTD | `TOTALYTD([Net Sales], DimDate[Date])` | Cumulative net sales from the start of the year |
| Net Sales MoM % | `DIVIDE([Net Sales] - CALCULATE([Net Sales], PREVIOUSMONTH(DimDate[Date])), CALCULATE([Net Sales], PREVIOUSMONTH(DimDate[Date])))` | Month-over-month growth rate |
| Net Sales 3M Avg | `AVERAGEX(DATESINPERIOD(DimDate[Date], MAX(DimDate[Date]), -3, MONTH), [Net Sales])` | Three-month rolling average, smooths monthly volatility |
| Invalid Discount Trx | `CALCULATE(COUNTROWS(Sales), Sales[flag_diskon_invalid] = TRUE)` | Data-quality check: transactions where discount ≥ gross amount (613 rows) |
