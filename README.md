<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F766E,50:0D9488,100:14B8A6&height=220&section=header&text=Online+Retail+Analytics&fontSize=52&fontColor=fff&animation=twinkling&fontAlignY=38&desc=2+Years+of+UK+E-Commerce+Data+%E2%80%A2+Python+%E2%80%A2+SQL+%E2%80%A2+Power+BI&descAlignY=62&descSize=18" />

<br/>

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=20&pause=1000&color=14B8A6&center=true&vCenter=true&width=750&lines=Real+UCI+Dataset+%E2%80%94+500%2C000%2B+UK+Transactions;Messy+Excel+%E2%86%92+Clean+SQLite+%E2%86%92+Power+BI;Returns+Separation+%2B+Customer+Attribution+Logic;Dual-Theme+Dashboard+for+Executive+Reporting)](https://github.com/yashxsainix)

<br/>

[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![SQLite](https://img.shields.io/badge/SQLite-07405E?style=for-the-badge&logo=sqlite&logoColor=white)](https://sqlite.org)
[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com)
[![Excel](https://img.shields.io/badge/Excel-217346?style=for-the-badge&logo=microsoftexcel&logoColor=white)](https://microsoft.com/excel)
[![UCI Dataset](https://img.shields.io/badge/Dataset-UCI_Online_Retail-0F766E?style=for-the-badge)](https://archive.ics.uci.edu/ml/datasets/Online+Retail)

<br/>

<img src="https://img.shields.io/badge/Transactions-500%2C000%2B-0F766E?style=flat-square&labelColor=0D1117" />
<img src="https://img.shields.io/badge/Time_Span-2_Years_(2009--2011)-0D9488?style=flat-square&labelColor=0D1117" />
<img src="https://img.shields.io/badge/Countries-38-14B8A6?style=flat-square&labelColor=0D1117" />
<img src="https://img.shields.io/badge/Pipeline-Excel_%E2%86%92_Python_%E2%86%92_SQL_%E2%86%92_BI-F2C811?style=flat-square&labelColor=0D1117" />

</div>

---

## 🛒 The Dataset

The UCI Online Retail dataset covers two years of transactions from a UK-based online retailer (2009–2011). It contains invoice records, product descriptions, quantities, unit prices, customer IDs, and country information across two separate Excel sheets — one per year.

It is a genuinely messy dataset:
- Null customer IDs that cannot be attributed
- Negative quantities representing returns
- Duplicate invoice records
- Two sheets that must be combined before any analysis is possible

Every cleaning decision in this pipeline is documented because it affects every downstream number.

---

## 🏗️ Pipeline Architecture

```mermaid
flowchart TD
    A["📊 Raw Source\nUCI Online Retail Excel\n2 sheets: 2009 + 2010\n~500K+ transactions"] -->|pandas read_excel| B

    subgraph CLEAN["🔧 Python: Clean & Engineer"]
        B["Combine Sheets"] --> C["Handle Nulls\n• Drop null CustomerID\n• Keep returns separate"]
        C --> D["Feature Engineering\n• Year, Month, Week, Quarter\n• DayOfWeek extraction\n• Column standardization"]
        D --> E["Sales/Returns Split\n• Negative qty = returns\n• Separate logic paths"]
    end

    E -->|sqlite3.connect + to_sql| F

    subgraph ANALYZE["📊 SQLite: 4 Structured Queries"]
        F["sales_summary\nRevenue trends over time"] 
        G["customer_analysis\nRFM-style customer metrics"]
        H["product_analysis\nTop products + seasonality"]
        I["country_analysis\nGeographic revenue split"]
    end

    F & G & H & I -->|CSV export| J["📊 Power BI\nDual-theme dashboard\nLight + dark mode"]

    style CLEAN fill:#0F766E,color:#fff,stroke:#0F766E
    style ANALYZE fill:#0D9488,color:#fff,stroke:#0D9488
    style J fill:#F2C811,color:#000,stroke:#F2C811
```

---

## 🔧 Cleaning Decisions That Matter

Every data project has cleaning decisions that the analyst makes. The wrong ones produce wrong dashboards. This project documents them explicitly.

<details>
<summary><b>❌ Why null CustomerIDs are dropped (not imputed)</b></summary>

Rows with null `CustomerID` cannot be attributed to any customer. That makes them useless for customer-level analysis, RFM modeling, and retention calculations — which is most of what this dashboard does. They are not imputed because there is no reliable basis for attribution.

If volume analysis is the only goal, they could be retained. They are not here.

</details>

<details>
<summary><b>📦 Why returns are separated, not dropped</b></summary>

Negative quantities represent returns or cancellations. Dropping them would overstate revenue. Mixing them with sales would understate it in the wrong places (e.g., a customer's total spend would look lower than their actual purchases).

The pipeline keeps them in a separate table and reports returns as a separate metric in the dashboard.

</details>

<details>
<summary><b>📅 Why date features are extracted in Python not Power Query</b></summary>

Extracting year, month, quarter, week, and day of week in Python before loading to SQLite means those fields are version-controlled, testable, and auditable — not buried in Power Query steps that are hard to review. Anyone reading the Python code can see exactly what time logic was applied.

</details>

---

## 🔍 The Four SQL Queries

```sql
-- 1. Sales summary — revenue trends over time
SELECT year, month, quarter,
       COUNT(DISTINCT invoice_no) as invoices,
       ROUND(SUM(quantity * unit_price), 2) as revenue
FROM sales
GROUP BY year, month, quarter
ORDER BY year, month;

-- 2. Top products by revenue
SELECT description,
       SUM(quantity) as total_units_sold,
       ROUND(SUM(quantity * unit_price), 2) as total_revenue
FROM sales
GROUP BY description
ORDER BY total_revenue DESC
LIMIT 20;

-- 3. Customer value analysis
SELECT customer_id,
       COUNT(DISTINCT invoice_no) as purchase_count,
       ROUND(SUM(quantity * unit_price), 2) as total_spend,
       MAX(invoice_date) as last_purchase
FROM sales
GROUP BY customer_id
ORDER BY total_spend DESC;

-- 4. Geographic revenue split
SELECT country,
       COUNT(DISTINCT customer_id) as customers,
       ROUND(SUM(quantity * unit_price), 2) as revenue
FROM sales
GROUP BY country
ORDER BY revenue DESC;
```

---

## 🚀 How to Run

```bash
git clone [repository-url]
pip install pandas openpyxl

# Step 1: Clean and engineer features
python etl_cleaning_sql_load.py

# Step 2: Run SQL analysis and export to CSV
python sql_analysis_export.py

# Step 3: Open Power BI and connect to the CSV outputs
# The dashboard includes both light and dark theme layouts
```

---

## 👤 Author

**Yashpal Saini** · [LinkedIn](https://linkedin.com/in/yash-saini-analyst) · [Portfolio](https://yashxsainix.github.io)

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F766E,50:0D9488,100:14B8A6&height=100&section=footer" />
