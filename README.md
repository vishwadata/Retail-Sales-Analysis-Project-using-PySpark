
# Retail Sales Analysis – Databricks Medallion Architecture Project using PySpark

An end-to-end data engineering project built on **Databricks**, implementing the **Medallion Architecture (Bronze → Silver → Gold)** using **PySpark** and **Delta Lake** to clean, transform, and model messy retail sales and customer data into analytics-ready Gold tables.

## 🏗️ Architecture

```
Raw CSV Files (messy data)
        │
        ▼
┌─────────────────┐
│   BRONZE LAYER   │  Raw ingestion, schema enforcement, ingestion metadata
└─────────────────┘
        │
        ▼
┌─────────────────┐
│   SILVER LAYER   │  Cleaning, standardization, deduplication, validation
└─────────────────┘
        │
        ▼
┌─────────────────┐
│    GOLD LAYER    │  Dimension & Fact tables, KPI-ready aggregates
└─────────────────┘
```

## 📂 Project Structure

| Layer | Tables | Purpose |
|---|---|---|
| **Bronze** | `bronze_orders`, `bronze_customers` | Raw data loaded as-is from CSV with enforced string schema + ingestion timestamp |
| **Silver** | `silver_orders`, `silver_customers` | Cleaned, validated, and standardized data |
| **Gold** | `gold_dim_customer`, `gold_fact_sales` | Dimensional model with fact/dimension tables and pre-aggregated KPIs |

## 🔧 What Each Layer Does

### 🥉 Bronze Layer
- Defines explicit schemas (`StructType`) for `orders` and `customers` raw files
- Reads raw, messy CSV files from a Databricks Volume
- Adds an `ingest_timestamp` column for lineage tracking
- Writes data as Delta tables with `overwrite` mode

### 🥈 Silver Layer

**Orders cleaning:**
- Parses multiple inconsistent date formats (`yyyy-MM-dd`, `dd-MM-yyyy`, `yyyy/MM/dd`, `dd/MM/yyyy`) using `try_to_date` + `coalesce`
- Standardizes category values (fixes typos like `ELECTRONIC` → `ELECTRONICS`, `FASHON` → `FASHION`)
- Removes duplicate orders based on `order_id`
- Handles null/invalid discount values and converts percentage strings (`"5%"`) into decimals (`0.05`)
- Removes records with negative quantities
- Casts `unit_price` to `DecimalType(10,2)`
- Recalculates `total_amount` using cleaned quantity, price, and discount

**Customers cleaning:**
- Trims whitespace across all string columns
- Removes duplicate customers, keeping the most recently ingested record (via `Window` + `row_number`)
- Standardizes `status` to `ACTIVE` / `INACTIVE` / `UNKNOWN`
- Cleans `loyalty_points` by stripping non-numeric characters and casting to integer
- Validates email format using regex and filters out invalid emails
- Parses inconsistent `date_of_birth` and `registration_date` formats
- Standardizes `gender` to `M` / `F` / `U`

### 🥇 Gold Layer
- Builds a **customer dimension table** (`gold_dim_customer`)
- Builds a **fact sales table** by joining cleaned orders with customers
- Enriches data with `Year`, `Month`, and `Day` extracted from `order_date`
- Aggregates KPIs by product, customer, category, and time:
  - Total revenue, total quantity, total orders
  - Average revenue, average quantity, average loyalty points
  - Min/max unit price
- Stores the final aggregated table as `gold_fact_Sales`

## 🛠️ Tech Stack

- **Databricks** (Notebooks, Unity Catalog, Volumes)
- **Apache Spark / PySpark**
- **Delta Lake**
- **Medallion Architecture** (Bronze–Silver–Gold)

## 📊 Sample KPIs Available in the Gold Layer

- Revenue by product, category, and customer
- Monthly/yearly sales trends
- Average order value and quantity per customer
- Customer loyalty points vs. spending behavior
- City/state-level sales performance

## 🚀 How to Run

1. Upload `retail_orders_messy.csv` and `retail_customers_messy.csv` to a Databricks Volume (e.g. `/Volumes/retail_project/raw_data/raw_file/`).
2. Create the `retail_project` catalog with `bronze`, `silver`, and `gold` schemas in Unity Catalog.
3. Run the notebook top to bottom in a Databricks workspace — each section (Bronze → Silver → Gold) writes its output as a Delta table before the next layer reads from it.
4. Query the final `retail_project.gold.gold_fact_sales` and `retail_project.gold.gold_dim_customer` tables for reporting/BI (e.g. Power BI, Databricks SQL dashboards).

## 📌 Notes

- The raw source files are intentionally "messy" (inconsistent date formats, typos, invalid emails, percentage-as-string discounts, negative quantities) to simulate real-world retail data quality issues.
- All transformations are designed to be idempotent — each layer can be re-run safely with `overwrite` mode.

## ✍️ Author

**Vishwa Bharath**

---

*Part of data engineering projects exploring the transition from traditional SAS/ETL pipelines to modern lakehouse architectures using PySpark, Delta Lake, and Databricks.*
