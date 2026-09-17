# Sales Data Pipeline — Medallion Architecture on Databricks

An end-to-end data engineering pipeline built on Databricks, implementing the **Bronze → Silver → Gold** medallion architecture to transform a raw multi-table sales dataset into business-ready analytics tables, feeding an interactive Power BI dashboard.

## Architecture

```
Excel Source (7 sheets)
       ↓
┌─────────────────┐
│  BRONZE LAYER    │  Raw ingestion → Delta tables
└─────────────────┘
       ↓
┌─────────────────┐
│  SILVER LAYER    │  Data quality, referential integrity,
│                  │  type casting, denormalized fact view
└─────────────────┘
       ↓
┌─────────────────┐
│  GOLD LAYER      │  Business KPIs & aggregated metrics
└─────────────────┘
       ↓
   Power BI Dashboard
```

## Data Model

A star schema sales dataset: one fact table plus six dimension tables covering customers, products (with category/subcategory hierarchy), regions, and dates.

| Table | Rows | Role |
|---|---|---|
| `factSalesTable` | 114,390 | Fact — sales transactions |
| `dimCustomerTable` | 9,999 | Dimension — customer demographics |
| `dimDateTable` | 1,188 | Dimension — calendar attributes |
| `dimRegionTable` | 655 | Dimension — geography |
| `dimProductTable` | 400 | Dimension — products |
| `dimProductSubcategoryTable` | 40 | Dimension — product subcategories |
| `dimProductCategoryTable` | 5 | Dimension — product categories |

## Bronze Layer — Raw Ingestion

- Reads all 7 sheets directly from the source Excel workbook using Spark's native Excel reader (`read_files` / `format("excel")`)
- Sanitizes column names and writes each sheet as a Delta table into the `workspace.bronze` schema
- Preserves data as-is (no cleaning) to keep an auditable raw copy

## Silver Layer — Data Quality & Transformation

- Profiled all 7 bronze tables and validated schemas
- **Primary key checks:** confirmed all dimension tables have unique keys
- **Referential integrity checks:** found **7,655 orphaned `CustomerKey`** records in the fact table (customer keys with no matching dimension record) — filtered these out, reducing the fact table from 114,390 → **63,738 valid transactions**
- Cast raw string columns to proper types (`int`, `decimal(10,2)`, `date`)
- Added calculated fields: `Revenue`, `Cost`, `Profit`
- Built a fully denormalized sales view by joining the fact table with all 6 dimensions (customer → region, product → subcategory → category, date), producing a single **28-column analytics-ready table**
- Saved all cleaned tables to the `workspace.silver` schema as Delta tables

## Gold Layer — Business Metrics & KPIs

Built pre-aggregated, business-facing tables on top of the silver denormalized view:

| Metric Table | What it answers |
|---|---|
| `total_sales_orders` | Overall business performance (single-row executive summary) |
| `sales_by_month` | Monthly revenue trends and seasonality (37 months) |
| `sales_by_country` / `sales_by_country_summary` | Geographic performance (46 country-region combinations) |
| `sales_by_product_category` / `_subcategory` / `_product` | Product hierarchy performance (158 unique SKUs) |
| `customer_summary` | Customer lifetime value, purchase frequency, and segmentation (RFM-style) |

### Headline Business Results

| Metric | Value |
|---|---|
| Total Revenue | **$450,314,774.86** |
| Total Cost | $264,342,354.18 |
| Total Profit | $185,972,420.68 |
| Profit Margin | **41.30%** |
| Total Orders | 11,157 |
| Unique Products Sold | 158 |

## Tech Stack

- **Databricks** — orchestration & compute
- **Apache Spark (PySpark)** — distributed data processing
- **Delta Lake** — table format for bronze/silver/gold layers
- **Power BI** — dashboard consuming the gold layer

## Repository Structure

```
├── Bronze_Layer_Data_Ingestion.ipynb
├── Silver_Layer_-_Data_Transformation.ipynb
├── Gold_Layer_-_Business_Metrics_&_KPIs.ipynb
└── README.md
```

---
*Author: Basmala Elhabshy*