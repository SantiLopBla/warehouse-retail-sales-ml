# Warehouse & Retail Sales — End-to-End ML Pipeline

![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)
![Databricks](https://img.shields.io/badge/Databricks-Free%20Edition-red?logo=databricks)
![Apache Spark](https://img.shields.io/badge/Apache%20Spark-4.1-orange?logo=apachespark)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-enabled-blue)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![License](https://img.shields.io/badge/License-MIT-green)

## Overview

End-to-end machine learning pipeline built on **Databricks** using the
**Medallion Architecture** (Bronze → Silver → Gold) on a real government
dataset of warehouse and retail sales transactions from Montgomery County, Maryland.

**Dataset:** 307,645 records · 11 columns · Updated monthly  
**Source:** [Montgomery County of Maryland — Warehouse and Retail Sales](https://catalog.data.gov/dataset/warehouse-and-retail-sales)  
**Publisher:** data.montgomerycountymd.gov  
**Issued:** July 6, 2017 · **Last modified:** May 5, 2026  
**Period covered:** June 2017 — September 2020  
**Categories:** WINE · BEER · LIQUOR · KEGS · NON-ALCOHOL

---

## Architecture

```
Raw CSV (307,645 rows)
        │
        ▼
┌───────────────────┐
│      BRONZE       │  Raw ingestion — Delta Table, zero transformations
│    ✅ Complete    │  307,645 rows · 9 columns
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│      SILVER       │  Cleaned, typed, enriched — 0 nulls
│    ✅ Complete    │  307,645 rows · 11 columns
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    EDA (Silver)   │  Exploratory Data Analysis · Distributions · Trends
│    ✅ Complete    │  5 analyses · Key findings documented
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│       GOLD        │  Business metrics · KPIs · ML-ready feature table
│    ⏳ Pending     │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    ML PIPELINE    │  Regression models · Experiment tracking
│    ⏳ Pending     │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    DASHBOARD      │  Business visualizations
│    ⏳ Pending     │
└───────────────────┘
```

---

## Project Structure

```
warehouse-retail-sales-ml/
│
├── notebooks/
│   ├── 01_bronze_ingestion.ipynb       ✅ Raw data ingestion
│   ├── 02_silver_transformation.ipynb  ✅ Cleaning & enrichment
│   ├── 03_silver_EDA.ipynb             ✅ Exploratory analysis
│   ├── 04_gold_metrics.ipynb           ⏳ Business aggregations
│   ├── 05_ml_pipeline.ipynb            ⏳ ML modeling
│   └── 06_dashboard.ipynb              ⏳ Visualizations
│
├── src/
│   ├── bronze.py                       ⏳ Reusable ingestion functions
│   ├── silver.py                       ⏳ Reusable transformation functions
│   ├── gold.py                         ⏳ Reusable aggregation functions
│   └── ml_pipeline.py                  ⏳ Reusable ML functions
│
├── docs/
│   └── architecture.md                 ⏳ Detailed architecture notes
│
├── .gitignore
├── LICENSE
└── README.md
```

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Apache Spark 4.1 | Distributed data processing |
| Delta Lake | Reliable storage layer (ACID transactions + time travel) |
| Databricks Serverless | Compute — no cluster management required |
| Python 3.12 | Core language |

---

## Dataset Schema — Silver Layer

| Column | Type | Description |
|--------|------|-------------|
| date | date | Transaction date |
| year | long | Year — kept for groupBy convenience |
| month | long | Month — kept for groupBy convenience |
| supplier | string | Distributor name (uppercase) |
| item_code | string | Product identifier |
| item_description | string | Full product name (uppercase) |
| item_type | string | Category: WINE, BEER, LIQUOR, KEGS, NON-ALCOHOL |
| retail_sales | double | Units sold at retail |
| retail_transfers | double | Units transferred between stores |
| warehouse_sales | double | Units sold from warehouse |
| total_sales | double | retail_sales + retail_transfers + warehouse_sales |

---

## Delta Tables

| Table | Layer | Rows | Columns | Status |
|-------|-------|------|---------|--------|
| `main.default.bronze_warehouse_sales` | Bronze | 307,645 | 9 | ✅ Ready |
| `main.default.silver_warehouse_sales` | Silver | 307,645 | 11 | ✅ Ready |
| `main.default.gold_monthly_sales` | Gold | — | — | ⏳ Pending |
| `main.default.gold_supplier_performance` | Gold | — | — | ⏳ Pending |

---

## Key Findings

- **BEER dominates by volume** — 7.6M units · avg 180.8 units per transaction
- **WINE dominates by frequency** — 187,640 transactions · avg 14 units each
- **Top 3 suppliers control 40.6% of total market volume** — Crown Imports, Miller Brewing, Anheuser Busch
- **Corona Extra** is the single best-selling product — 352,574 units
- **All top 20 products are BEER** — no other category appears in the top 20
- **BEER is 85% warehouse channel** (B2B) · **LIQUOR is 47% retail** (consumer)
- **LIQUOR has 47% inter-store transfer rate** — highest inventory rebalancing of any category
- Dataset covers **24 months across 2017–2020** — not continuous, significant gaps in 2018 and 2020
- **2019 is the most reliable year** — 11 consecutive months of complete data
- All null values resolved in Silver — **0 nulls across all 11 columns**

---

## Author

**Santiago López Blanco**  
Data Science Engineering Student — Universidad Fidélitas, Costa Rica

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://www.linkedin.com/in/santiago-lópez-blanco-420886342)
[![GitHub](https://img.shields.io/badge/GitHub-SantiLopBla-black?logo=github)](https://github.com/SantiLopBla)

---

## Status

This project is actively under development. New notebooks and results
will be added as each phase is completed.

> Last updated: May 2026
