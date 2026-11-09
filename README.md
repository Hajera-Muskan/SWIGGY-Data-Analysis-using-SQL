# Swiggy Growth & Operations Analytics (SQL)

SQL analysis of a simulated Swiggy food-delivery dataset: revenue and discounting, customer behaviour, delivery quality and campaign performance. Queries were written in Databricks SQL (with a pandas section for comparison) and the results were visualised in a Zoho Analytics dashboard.

## Data model

A star schema: one fact table and five dimensions.

| Table | Contents |
|---|---|
| `fact_transactions` | One row per order: gross/discount/net amounts, coupon, membership, campaign, surge, rain and delivery fields |
| `dim_customer` | Name, gender, age, home city, signup date, membership tier |
| `dim_geo` | City, state, pincode |
| `dim_restaurant_product` | Restaurant, product, cuisine tag, list price |
| `dim_coupon`, `dim_campaign` | Coupon rules and campaign metadata |

Scope of the full analysis: 158,400 orders, 6,000 customers, Bangalore and Hyderabad, 27 Dec 2024 to 27 Dec 2025. Column definitions are in `data/data_dictionary.xlsx`.

> The data is simulated. `data/swiggy_fact_transactions.xlsx` is a 9,483-row slice of the fact table, so numbers computed from it will not match the full-data results in `results/` and `dashboard/`.

## What was analysed (29 questions)

- **A. Warm-up:** data preview, date range, orders by city, coupon share, average delivery time
- **B. Core:** monthly revenue by city, coupon and membership discount share, top customers, monthly repeat rate, delivery success and rain impact, slowest deliveries, campaign CTR and order value
- **C. Dimension-heavy:** joins across all dimensions, orphan-record checks, top cuisines per city and month, restaurant leaderboard, coupon and campaign performance, geo and customer segments
- **D. Set operations:** customer overlap between cities with `UNION`, `INTERSECT` and `EXCEPT`

**SQL techniques:** conditional aggregation, `NULLIF`, date functions, CTEs, scalar subqueries with `percentile_approx`, inner and left joins, anti-joins, `ROW_NUMBER`, set operations. pandas: `merge`, `groupby`, `cumcount`.

## Selected findings (full data)

- Rain adds about 8.5 minutes to delivery time in both cities.
- 27.4% of orders use a coupon; WELCOME50 accounts for about 40% of total coupon discount.
- Delivery success is about 96% across cities and devices.
- Campaign CTR is about 21-22% across the four campaigns.
- Orphan checks found no fact rows without a matching dimension record.

## Repository layout

```
notebooks/   SQL_Mini_Project_SWIGGY.ipynb   Databricks notebook (SQL and pandas)
results/     Outputs_Mini_Project_Swiggy_V2.xlsx   One sheet per question, with the SQL
dashboard/   Swiggy_Dashboard_Zoho_Analytics.pdf
data/        Dimension tables, fact-table slice and data dictionary
```

## Running the notebook

1. In Databricks, create the schema `workspace.swiggy` and load each file in `data/` as a table named `swiggy_<name>` (for example `swiggy_dim_customer`).
2. Cells 18-26 query `swiggy_fact_transactions_v_2`, a later version of the fact table that is not included here. To run them on the included data, point them at `swiggy_fact_transactions`.

## Data notes

- Dec 2024 holds only 5 days of data (27-31 Dec), so grouping by calendar month alone merges two different Decembers.
- `net_amount` has a floor of 40, so it can exceed `gross_amount - total_discount_amount` on very small orders.
- `cuisine_tag` is not consistent with product names, so cuisine rankings are illustrative only.
- No customer ordered in both cities, so the `INTERSECT` query returns no rows.
