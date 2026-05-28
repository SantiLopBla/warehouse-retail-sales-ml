# Warehouse & Retail Sales — End-to-End ML Pipeline

![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)
![Databricks](https://img.shields.io/badge/Databricks-Free%20Edition-red?logo=databricks)
![Apache Spark](https://img.shields.io/badge/Apache%20Spark-4.1-orange?logo=apachespark)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-enabled-blue)
![MLflow](https://img.shields.io/badge/MLflow-Tracking-blue?logo=mlflow)
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
│    ✅ Complete    │  Best model: LightGBM · R² 96.84% · MAE $276.81
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
│   ├── 05_ml_model.ipynb               ✅ ML modeling
│   └── 06_dashboard.ipynb              ⏳ Visualizations
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
| Databricks Free Edition | Compute — no cluster management required |
| MLflow | Experiment tracking and model registry |
| Python 3.12 | Core language |
| scikit-learn | ML models and TimeSeriesSplit cross-validation |
| LightGBM | Gradient boosting — best model |
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

### Objective

Predict `next_month_sales` for each `item_type + supplier` combination using lag features, rolling averages, and seasonality encoding.

### Train / Test Split

| Set | Years | Rows |
|-----|-------|------|
| Train + CV | 2017, 2018, 2019 | 7,101 |
| Test | 2020 | 1,118 |

Cross-validation uses `TimeSeriesSplit` with 5 folds — validation always uses future data relative to training. The 2020 test set coincides with the COVID-19 pandemic, producing honest metrics under real-world anomaly conditions.

### Model Results

| Model | CV MAE | CV MAE ± | Test MAE | Test RMSE | Test R² |
|-------|--------|----------|----------|-----------|---------|
| **LightGBM** | $255.03 | ±$254.55 | **$276.81** | **$1,312.87** | **96.84%** |
| Random Forest | $118.18 | ±$57.73 | $284.64 | $1,531.97 | 95.70% |
| XGBoost | $116.80 | ±$61.10 | $305.23 | $1,820.86 | 93.93% |
| Linear Regression | $346.56 | ±$245.58 | $359.67 | $2,038.51 | 92.39% |

**Best model: LightGBM** — wins on Test MAE, Test RMSE, and R². XGBoost achieved the lowest CV MAE but its test performance reveals mild overfitting to the training distribution.

### Feature Importance (LightGBM)

| Feature | Importance |
|---------|------------|
| lag_2_sales | 41.91% |
| lag_1_sales | 30.47% |
| supplier_tier_enc | 13.76% |
| rolling_3m_avg | 11.34% |
| All others combined | 2.52% |

Recent sales history accounts for 71% of all model decisions. The strongest predictor of next month's sales is what was sold in the last two months.

### Performance by Segment

**By product type:**

| Segment | Test MAE | Test R² | Notes |
|---------|----------|---------|-------|
| STR_SUPPLIES | $49.69 | 69.54% | Low volume, stable |
| KEGS | $50.67 | -4.18% | Model worse than average — event-driven demand |
| NON-ALCOHOL | $130.45 | 49.68% | Insufficient features for this segment |
| WINE | $145.36 | 90.77% | Reliable |
| LIQUOR | $172.84 | 95.42% | Reliable |
| BEER | $907.23 | 96.88% | High MAE driven by COVID shock in March 2020 |

**By supplier tier:**

| Tier | Test MAE | Test R² | Notes |
|------|----------|---------|-------|
| rest | $112.22 | 89.47% | Low and stable volumes |
| top15 | $1,480.80 | 87.51% | Hardest segment to forecast |
| top3 | $3,160.43 | 97.27% | Large MAE in dollars but strong pattern capture |

### Model Registry

The trained LightGBM model is registered in the MLflow Model Registry under `workspace.default.lightgbm_sales_forecaster`. Encoders are saved as MLflow artifacts alongside the model for reproducible inference.

---

## Key Findings

### EDA
- **BEER dominates by volume** — 7.67M units · 62.8% of total market · avg 180.8 units per transaction
- **WINE dominates by frequency** — 187,640 transactions · avg 14.06 units each
- **Top 3 suppliers control 41.2% of total market volume** — Crown Imports (14.9%), Anheuser Busch, Miller Brewing
- **395 distinct suppliers** — extreme concentration in top 3
- **Corona Extra** is the single best-selling product — 352,574 units · present all 24 months
- **All top 20 products are BEER** — no other category appears in the top 20
- **BEER is 85% warehouse channel** (B2B bulk) · **LIQUOR is 94% store-facing** (retail + transfers)

### ML Pipeline
- **LightGBM generalizes best** — wins on all test metrics despite weaker cross-validation scores
- **Lag features drive 71% of predictions** — recent sales history is the strongest signal by far
- **KEGS is the hardest segment** — negative R² indicates the current feature set is insufficient
- **March 2020 is the largest error month** — COVID-19 demand shock caused a -12.79% underestimation that no model trained on pre-pandemic data could anticipate
- **top15 suppliers are the hardest tier to forecast** — moderate volume combined with high variability produces the weakest R² across tiers

---

## Author

**Santiago López Blanco**  
Data Science Engineering Student — Universidad Fidélitas, Costa Rica

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://www.linkedin.com/in/santiago-l%C3%B3pez-blanco-ds)

---

> Last updated: May 2026
