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
**Categories:** WINE · BEER · LIQUOR · KEGS · NON-ALCOHOL · STR_SUPPLIES

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
│    ✅ Complete    │  2 tables · 9,980 + 8,219 rows
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    ML PIPELINE    │  Demand forecasting · 4 models · TimeSeriesSplit CV
│    ⏳ In Progress │
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
│   ├── 04_gold_layer.ipynb             ✅ Business metrics & ML features
│   ├── 05_ml_model.ipynb               ⏳ ML modeling
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
| scikit-learn | ML models and TimeSeriesSplit cross-validation |
| LightGBM | Gradient boosting — fast and memory-efficient |
| XGBoost | Optimized gradient boosting |

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
| item_type | string | Category: WINE, BEER, LIQUOR, KEGS, NON-ALCOHOL, STR_SUPPLIES |
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
| `main.default.gold_business_metrics` | Gold | 9,980 | 12 | ✅ Ready |
| `main.default.gold_ml_features` | Gold | 8,219 | 14 | ✅ Ready |

---

## ML Pipeline

| Model | Type | CV Strategy |
|-------|------|-------------|
| Linear Regression | Baseline | TimeSeriesSplit (5 folds) |
| Random Forest | Bagging | TimeSeriesSplit (5 folds) |
| LightGBM | Gradient Boosting | TimeSeriesSplit (5 folds) |
| XGBoost | Optimized Boosting | TimeSeriesSplit (5 folds) |

**Split:** Train+CV = 2017–2019 (7,101 rows) · Test = 2020 (1,118 rows)  
**Metrics:** MAE · RMSE · R²  
**Note:** 2020 test set coincides with COVID-19 — metrics reflect real-world anomaly conditions

---

## Key Findings

- **BEER dominates by volume** — 7.67M units · 62.8% of total market · avg 180.8 units per transaction
- **WINE dominates by frequency** — 187,640 transactions · avg 14.06 units each
- **Top 3 suppliers control 41.2% of total market volume** — Crown Imports (14.9%), Anheuser Busch, Miller Brewing
- **395 distinct suppliers** — extreme concentration in top 3 out of 395
- **Corona Extra** is the single best-selling product — 352,574 units · present all 24 months
- **All top 20 products are BEER** — no other category appears in the top 20
- **BEER is 85% warehouse channel** (B2B bulk) · **LIQUOR is 94% store-facing** (retail + transfers)
- **2019 is the only reliable year** — 11 consecutive months · 5.53M units
- **2018 has only 2 months** of data · **2020 has 4 non-consecutive months**
- All null values resolved in Silver — **0 nulls across all 11 columns**

---

## Author

**Santiago López Blanco**  
Data Science Engineering Student — Universidad Fidélitas, Costa Rica

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://www.linkedin.com/in/santiago-l%C3%B3pez-blanco-ds)

---

## Status

This project is actively under development. New notebooks and results
will be added as each phase is completed.

> Last updated: May 2026
