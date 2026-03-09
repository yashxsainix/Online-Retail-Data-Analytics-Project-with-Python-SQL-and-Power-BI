# Online Retail Analytics

An end-to-end analytics pipeline built on two years of real UK e-commerce transaction data. The project goes from raw Excel files to a structured SQLite database to a dual-theme Power BI dashboard, with every transformation step documented and reproducible.

---

## The Dataset

The [UCI Online Retail dataset](https://archive.ics.uci.edu/ml/datasets/Online+Retail) covers transactions from a UK-based online retailer between 2009 and 2011. It contains invoice records, product descriptions, quantities, prices, customer IDs, and country information across two Excel sheets — one per year.

It is a genuinely messy dataset. There are null customer IDs, negative quantities representing returns, duplicate records, and two separate sheets that need to be combined before any analysis is possible. The cleaning decisions made here are documented because they affect every downstream number.

---

## Pipeline Overview

```
Raw Excel (2 sheets, 2009-2011)
        |
        v
Python: Combine, Clean, Engineer Features
        |
        v
SQLite Database
        |
        v
SQL Analysis (4 structured queries)
        |
        v
CSV Exports
        |
        v
Power BI Dashboard (light + dark theme)
```

The choice to route through SQLite rather than loading directly into Power BI was deliberate. It keeps the cleaning and transformation logic in Python and SQL where it is version-controllable and auditable, rather than buried in Power Query steps that are harder to review.

---

## Part 1 — Cleaning and Feature Engineering

The two Excel sheets are combined into a single DataFrame and passed through the following steps:

**Null handling:** Rows with null customer IDs are dropped. These represent transactions that cannot be attributed to any customer, which makes them useless for the customer-level analysis this project focuses on.

**Deduplication:** Exact duplicates removed after the combine step.

**Date engineering:** `InvoiceDate` is cast to datetime. Year, month, week number, quarter, and day of week are extracted as separate columns. These make time-based SQL grouping significantly faster and more readable downstream.

**Column standardization:** All column names converted to lowercase with underscores for SQL compatibility.

**Separation of sales and returns:** Transactions with negative quantities are returns. They are not dropped — they are kept in the database and queried separately. Mixing them into the sales analysis without accounting for them would inflate revenue figures.

---

## Part 2 — SQL Analysis

Four structured queries produce the datasets that feed Power BI. Each one is documented here because the logic matters for interpreting the dashboard.

**sales_summary**
Clean sales transactions only (quantity > 0). Includes a calculated `total_revenue` column (quantity x price) and all date dimensions. This is the primary fact table for revenue and volume analysis.

**products_by_quantity**
Products ranked by total units sold, excluding returns. Used for the product performance visuals.

**customer_activity**
One row per customer, showing total spend, last purchase date, and a count of distinct active months. The `active_months` field is what drives the customer type segmentation downstream.

**returns_summary**
Returns only (quantity < 0). Separate from sales so return rates can be analyzed without contaminating revenue figures. Includes total return value and return quantity per transaction.

---

## Part 3 — Power BI Dashboard

### DAX Measures

```dax
Total Revenue = SUM(sales_summary[total_revenue])
Total Quantity Sold = SUM(sales_summary[total_quantity])
Total Invoices = DISTINCTCOUNT(sales_summary[invoice])
Revenue per Customer = [Total Revenue] / [Number of Customers]
Average Order Value = [Total Revenue] / [Total Invoices]
Total Returns Value = SUM(returns_summary[total_returns_value])
Returns Percentage = DIVIDE([Total Returns Value], [Total Revenue])
Number of Customers = DISTINCTCOUNT(customer_activity[customer_id])
```

### Calculated Column — Customer Type

```dax
Customer Type =
IF(
    [active_months] <= 2,
    "Inactive",
    IF(
        [active_months] <= 5,
        "Normal",
        "Active"
    )
)
```

This column classifies customers by engagement level and is used as a cross-report slicer throughout the dashboard. It is the single most useful filter for understanding revenue concentration.

### Dashboard Sections

**KPI strip (top):** Total Revenue, Quantity Sold, Revenue per Customer, Returns Percentage, Returns Count, Returns Value. Four of the KPIs have tooltip charts showing the metric over time — a small design decision that lets someone spot a trend without having to navigate to a separate page.

**Revenue over time:** Monthly line chart across the full two-year period. Q4 spikes visible in both years, consistent with UK holiday shopping patterns.

**Customer insights table:** Customer ID, country, active months, customer type, average spend, total revenue. Sortable. Most useful sorted by total revenue to immediately see who the top accounts are.

**Returns section:** Three visuals — returns value over time, returns by country as a treemap, and revenue vs return count by product. The product visual specifically is useful for identifying items with high revenue but disproportionately high return rates.

**Active months vs revenue chart:** Shows the relationship between customer loyalty and revenue contribution. The curve is not linear — the jump from Normal to Active customers is where most of the revenue lives.

**AOV and average price over time:** Tracks order value and unit price week by week. Useful for spotting pricing anomalies and seasonal spending shifts.

**Slicers:** Customer Type, Year, Country. All three affect every visual on the page simultaneously.

### Two Themes

A light theme and a dark theme are both included. The light version works for executive presentations and printed reports. The dark version works for large screens and internal team reviews. The data and logic are identical — only the visual treatment differs.

A recorded dashboard walkthrough is available [here](https://www.youtube.com/watch?v=tJZT9xYB1fI&t=5s).

---

## Key Findings

| Finding | Detail |
|---|---|
| Revenue concentration | Top 27% of customers (Active) generated 80% of revenue — $13.4M of $17.4M |
| Customer base breakdown | 1,597 Active / 5,942 total customers |
| Top product by revenue | Regency Cakestand 3 Tier — over $277K |
| Top country by revenue | United Kingdom — $14M |
| Top country by returns | United Kingdom — $906.7K (largest customer base) |
| Overall return rate | Under 20% across all customer segments |
| Inactive customer behavior | Highest return rates, lowest revenue contribution |
| AOV pattern | Seasonal peaks driven by multi-item purchases, stable average unit price |

---

## How to Run

**Requirements:**
- Python 3.8+
- pandas, openpyxl, sqlite3 (standard library)
- Power BI Desktop (for the .pbix file)
- The raw dataset from [UCI ML Repository](https://archive.ics.uci.edu/ml/datasets/Online+Retail)

**Steps:**

```bash
# Install dependencies
pip install pandas openpyxl

# Step 1: Clean the data and load to SQLite
python etl_cleaning_sql_load.py

# Step 2: Run SQL analysis and export CSVs
python sql_analysis_export.py

# Step 3: Open Power BI and connect to the CSV files
# Load sales_summary.csv, customer_activity.csv,
# products_by_quantity.csv, returns_summary.csv
```

---

## Repository Structure

```
Online-Retail-Analytics/
├── etl_cleaning_sql_load.py    Cleaning, feature engineering, SQLite load
├── sql_analysis_export.py      SQL queries and CSV exports
├── dax_formulas                All DAX measures and calculated columns
└── README.md
```

Note: The raw Excel files and .pbix dashboard file are not included in the repo due to file size. The dataset is publicly available at the UCI link above.

---

## Author

**Yash Saini**
MS in Business Analytics, Baruch College
[github.com/yashxsainix](https://github.com/yashxsainix) · [linkedin.com/in/yash-saini-analyst](https://linkedin.com/in/yash-saini-analyst)
