# End-to-End SQL Data Warehouse and Sales Analytics

**Turning fragmented CRM and ERP data into a single, trusted source of truth, then using it to answer real business questions about customers, products, and revenue.**

| | |
|---|---|
| **Role demonstrated** | Data Analyst / BI Analyst / Analytics Engineer (entry level) |
| **Core skills** | SQL (T-SQL), ETL, Data Modeling, Data Quality, Business Analytics |
| **Architecture** | Medallion (Bronze, Silver, Gold), Star Schema |
| **Stack** | SQL Server, SSMS, draw.io, Git |
| **Author** | Yasir Sheikh |

---

## 30-Second Summary 

| Question | Answer |
|---|---|
| **What problem does it solve?** | Sales, customer, and product data sat in separate CRM and ERP systems with inconsistent formats, duplicates, and missing values, making reliable reporting impossible. |
| **What did I build?** | A three-layer data warehouse that ingests raw CSVs, cleans and standardizes them, and publishes a star schema ready for analysis. |
| **What can it answer?** | Top customers, best-selling products, revenue trends, category contribution, and period-over-period performance. |
| **What does it prove about me?** | I can take messy source data from raw files to business insights using SQL, with documented design decisions and validation checks. |

---

## Table of Contents

1. [Business Problem](#business-problem)
2. [Solution Overview](#solution-overview)
3. [Key Results and Insights](#key-results-and-insights)
4. [Architecture](#architecture)
5. [Data Model](#data-model)
6. [ETL and Data Cleaning](#etl-and-data-cleaning)
7. [Data Quality Framework](#data-quality-framework)
8. [Analytics and Sample Queries](#analytics-and-sample-queries)
9. [Design Decisions and Challenges](#design-decisions-and-challenges)
10. [Skills Demonstrated](#skills-demonstrated)
11. [Repository Structure](#repository-structure)
12. [How to Run](#how-to-run)
13. [Future Improvements](#future-improvements)
14. [About the Author](#about-the-author)

---

## Business Problem

A company runs its operations across two systems: a **CRM** (customers, products, sales) and an **ERP** (extra customer and product attributes). The data has these problems:

| Problem | Business Impact |
|---|---|
| Same customer appears in both systems with different formats | Inaccurate customer counts and segmentation |
| Duplicates, nulls, and inconsistent codes | Unreliable KPIs and mistrust in reports |
| No single analytical model | Analysts spend time cleaning data instead of finding insights |
| Dates and numbers stored as text | Time-based analysis is error-prone |

**Objective:** Build a centralized warehouse that integrates both sources, enforces data quality, and exposes a clean model for reporting.

---

## Solution Overview

| Stage | What Happens | Result |
|---|---|---|
| 1. Ingest | Raw CSVs loaded as-is into the Bronze layer | Full traceability to source |
| 2. Cleanse | Silver layer fixes duplicates, nulls, types, and codes | Consistent, validated data |
| 3. Model | Gold layer joins sources into a star schema | Analysis-ready fact and dimension views |
| 4. Analyze | SQL reports built on the Gold layer | Answers to business questions |

---

## Key Results and Insights

> Replace the bracketed values with your real numbers. Specific figures are what make this section stand out.

| Metric | Result |
|---|---|
| Source files integrated | [X] CSV files across [2] systems |
| Raw records processed | [X,XXX] rows |
| Data quality issues resolved | [X] types of issues (duplicates, nulls, invalid dates, inconsistent codes) |
| Final analytical objects | [X] dimension views and [X] fact view |
| Reports built | [X] SQL analyses |

**Business insights uncovered**

| # | Insight | Supporting Analysis |
|---|---|---|
| 1 | [e.g., Top X% of customers generate Y% of revenue] | Customer segmentation query |
| 2 | [e.g., Category A contributes Z% of total sales] | Part-to-whole analysis |
| 3 | [e.g., Sales peaked in month/year X and declined afterward] | Time-series trend analysis |
| 4 | [e.g., Product Y has the highest revenue but lowest margin] | Product performance query |

---

## Architecture

| Layer | Purpose | Load Type | Object Type |
|---|---|---|---|
| **Bronze** | Raw, unmodified source data | Full load (truncate and insert) | Tables |
| **Silver** | Cleansed, standardized, deduplicated data | Full load | Tables |
| **Gold** | Business-ready star schema | Virtual | Views |

```
  CRM (CSV) ─┐
             ├──> BRONZE (raw) ──> SILVER (clean) ──> GOLD (star schema) ──> Reports
  ERP (CSV) ─┘
```

> Add your diagram: `![Architecture](docs/data_architecture.png)`

---

## Data Model

Gold layer star schema:

| Object | Type | Grain | Purpose |
|---|---|---|---|
| `dim_customers` | Dimension | One row per customer | Combines CRM and ERP customer attributes |
| `dim_products` | Dimension | One row per product | Product, category, and cost details |
| `fact_sales` | Fact | One row per order line | Sales, quantity, price, linked by surrogate keys |

```
   dim_customers          dim_products
          \                    /
           +---- fact_sales ---+
```

> Add your data model diagram: `![Data Model](docs/data_model.png)`

---

## ETL and Data Cleaning

| Issue Found | Technique Used | Layer |
|---|---|---|
| Duplicate customer records | `ROW_NUMBER()` to keep the latest record per key | Silver |
| Extra whitespace in text | `TRIM()` | Silver |
| Coded values (gender, marital status, country) | `CASE` mapping to readable labels | Silver |
| Missing or invalid values | `COALESCE` / `ISNULL` and rule-based defaults | Silver |
| Dates stored as integers or text | Safe type casting with validation | Silver |
| Inconsistent sales values | Recalculated from quantity and price | Silver |
| Integrating two sources | Joins on business keys, with conflict-resolution rules | Gold |
| Stable keys for modeling | Surrogate keys using `ROW_NUMBER()` | Gold |

---

## Data Quality Framework

| Check | Why It Matters |
|---|---|
| Primary key uniqueness | Prevents double counting |
| Null checks on required fields | Ensures completeness |
| Whitespace and format checks | Ensures consistency |
| Referential integrity (fact to dimensions) | Prevents orphan records |
| Date logic (order date before ship date) | Ensures business validity |
| Row count reconciliation across layers | Confirms nothing was lost in transformation |

---

## Analytics and Sample Queries

| Analysis | Technique | Business Question |
|---|---|---|
| Sales trend over time | `GROUP BY`, date functions | How is revenue changing monthly and yearly? |
| Running totals and moving averages | Window functions | Is growth accelerating or slowing? |
| Period-over-period performance | `LAG()`, `LEAD()` | How does each period compare to the previous one? |
| Product performance vs. average | Window aggregates | Which products over- or under-perform? |
| Part-to-whole contribution | Aggregations, percentages | Which categories drive revenue? |
| Customer segmentation | `CASE`, grouping | Who are the high-value and at-risk customers? |

**Example query**

```sql
-- Revenue contribution by product category
SELECT
    category,
    SUM(sales_amount) AS total_sales,
    ROUND(SUM(sales_amount) * 100.0 / SUM(SUM(sales_amount)) OVER (), 2) AS pct_of_total
FROM gold.fact_sales f
LEFT JOIN gold.dim_products p
    ON f.product_key = p.product_key
GROUP BY category
ORDER BY total_sales DESC;
```

> Add screenshots of your query outputs or a dashboard here.

---

## Design Decisions and Challenges

| Decision or Challenge | Approach | Reasoning |
|---|---|---|
| Why Medallion architecture? | Separate raw, clean, and business layers | Traceability, easier debugging, reusable clean data |
| Gold as views, not tables | Views over Silver | Always current, no extra storage or load step |
| Two sources disagree on a value | Defined a master source and fallback rules | Avoids ambiguous results and documents the logic |
| Reloading data safely | Truncate-and-insert in stored procedures | Repeatable and idempotent loads |
| Keeping the project maintainable | Naming conventions and a data catalog in `docs/` | Anyone can understand and extend it |

---

## Skills Demonstrated

| Skill Category | Specifics |
|---|---|
| **SQL** | Joins, CTEs, subqueries, window functions, aggregations, views, stored procedures |
| **Data Warehousing** | Medallion architecture, star schema, fact and dimension design |
| **ETL** | Extraction, cleansing, transformation, and loading |
| **Data Quality** | Validation rules, reconciliation, profiling |
| **Analytics** | KPI design, trend, segmentation, and contribution analysis |
| **Documentation** | Architecture diagrams, data catalog, naming conventions |
| **Tools** | SQL Server, SSMS, draw.io, Git, GitHub |

---

## Repository Structure

```
sql-data-warehouse-project/
├── datasets/     # Source CSV files (CRM and ERP)
├── docs/         # Diagrams, data catalog, naming conventions
├── scripts/      # Database setup plus bronze, silver, and gold SQL
├── tests/        # Data quality and validation queries
└── README.md
```

---


## Future Improvements

| Enhancement | Value |
|---|---|
| Power BI dashboard on the Gold layer | Interactive visual reporting for stakeholders |
| Incremental loading | Process only new or changed records |
| Scheduled automation | Remove manual execution |
| Logging and error handling | Monitoring and easier debugging |

---

## About the Author

I am B.Tech Computer Science and Engineering student (2023 to 2027) at G H Raisoni University, Amravati, seeking entry-level **Data Analyst / BI Analyst** roles.

| | |
|---|---|
| GitHub | [mohdyasir5155](https://github.com/mohdyasir5155) |
| LinkedIn | [https://www.linkedin.com/in/mohd-yasir-sheikh/?isSelfProfile=true] |
| Email | [mohdyasir5155@gmail.com] |
