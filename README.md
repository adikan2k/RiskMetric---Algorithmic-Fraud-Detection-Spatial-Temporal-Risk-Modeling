# 🛡️ RiskMetric — Algorithmic Fraud Detection & Spatial-Temporal Risk Modeling

> A zero-cost, high-performance **Trust Engine** that identifies complex fraud archetypes across **1M+ synthetic banking transactions** with sub-second inference speed.

![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python)
![DuckDB](https://img.shields.io/badge/DuckDB-OLAP-yellow?logo=duckdb)
![dbt](https://img.shields.io/badge/dbt--core-Pipeline-orange?logo=dbt)
![Streamlit](https://img.shields.io/badge/Streamlit-Dashboard-red?logo=streamlit)

---

### 📊 Executive Overview
![RiskMetric Landing Page](Riskmetric%20Landing%20page.png)
*The executive dashboard provides real-time KPI tracking, fraud archetype distributions, and spatial-temporal risk metrics across all processed transactions.*

### 🗺️ Geographic & Behavioral Analysis
![RiskMetric Geographic Analysis](Riskmetric%202.png)
*Detailed spatial analysis mapping impossible travel hotspots, alongside velocity spike and behavioral drift detection metrics to identify compromised accounts.*

### 🚨 Operational Triage Queue
![RiskMetric Triage Queue](Triage%20Queue.png)
*An interactive alert management system allowing fraud analysts to filter by risk tier, review evidence (e.g., ground speed, Z-scores), and disposition cases in real-time.*

---

## Architecture — Medallion System

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Actions CI/CD                      │
├──────────┬──────────────────┬───────────────┬───────────────┤
│  BRONZE  │      SILVER      │     GOLD      │   INTERFACE   │
│          │                  │               │               │
│ generator│ Impossible Travel│ Risk Scores   │  Streamlit    │
│   .py    │ (Haversine/GPS)  │ (Composite)   │  Dashboard    │
│          │                  │               │               │
│ 1M+ raw  │ Velocity Spikes  │ User Risk     │  Real-time    │
│ records  │ (60s windows)    │ Profiles      │  Watchdog     │
│          │                  │               │               │
│ Parquet  │ Behavioral Drift │ Fraud         │  SQL Explorer │
│          │ (Z-Score/30d MA) │ Attribution   │               │
├──────────┼──────────────────┼───────────────┼───────────────┤
│  Faker   │     DuckDB       │   dbt-core    │   Plotly      │
│  PyArrow │   + dbt macros   │  + dbt tests  │   + Geo Maps  │
└──────────┴──────────────────┴───────────────┴───────────────┘
```

## Fraud Archetypes Detected

| Archetype | Method | Threshold |
|---|---|---|
| **Impossible Travel** | Haversine distance + ground speed between consecutive transactions | > 500 mph |
| **Velocity Spikes** | 60-second sliding window transaction count | ≥ 10 txns / 60s |
| **Behavioral Drift** | Z-Score against 30-day rolling average | \|Z\| > 3σ |

## Tech Stack ($0 Cost)

- **Engine**: DuckDB — vectorized, in-process OLAP
- **Storage**: Apache Parquet — columnar, compressed
- **Pipeline**: dbt-core — modular, tested, version-controlled SQL
- **Orchestration**: GitHub Actions — CI/CD on every push
- **Interface**: Streamlit + Plotly — executive-level dashboard

## Quick Start

### 1. Install Dependencies

```bash
pip install -r requirements.txt
```

### 2. Run Full Pipeline (Bronze → Silver → Gold)

```bash
python run_pipeline.py
```

This will:
- Generate 1M+ synthetic transactions with injected fraud
- Run all dbt models (Silver enrichment + Gold aggregation)
- Run dbt data integrity tests
- Export Gold tables to Parquet
- Generate dbt documentation and data lineage graph
- Print a summary report with model evaluation metrics

### 3. Launch Dashboard

```bash
streamlit run dashboard/app.py
```

### 4. View Data Lineage (dbt Docs)

```bash
dbt docs serve --profiles-dir riskmetric --project-dir riskmetric
```

### Manual Steps (Optional)

```bash
# Generate data only
python generator.py

# Run dbt models only (with full refresh for incremental models)
cd riskmetric
dbt run --profiles-dir . --project-dir . --full-refresh
dbt test --profiles-dir . --project-dir .
dbt docs generate --profiles-dir . --project-dir .
```

## Project Structure

```
Banking Sentinel/
├── generator.py                    # Bronze: 1M+ synthetic transaction generator
├── run_pipeline.py                 # Full pipeline orchestrator
├── config.yml                      # Centralized thresholds & calibration config
├── requirements.txt                # Python dependencies
├── data/                           # Generated data (gitignored)
│   ├── raw_transactions.parquet    # Bronze layer
│   ├── user_profiles.parquet       # Bronze layer
│   └── riskmetric.duckdb           # DuckDB warehouse
├── riskmetric/                     # dbt project
│   ├── dbt_project.yml             # dbt config + vars for threshold calibration
│   ├── profiles.yml
│   ├── macros/
│   │   └── haversine.sql           # Haversine distance macro
│   └── models/
│       ├── staging/                # Raw → Staged
│       │   ├── stg_transactions.sql
│       │   └── stg_user_profiles.sql
│       ├── silver/                 # Fraud detection (incremental)
│       │   ├── silver_impossible_travel.sql
│       │   ├── silver_velocity_spikes.sql
│       │   └── silver_behavioral_drift.sql
│       └── gold/                   # Curated risk models
│           ├── gold_risk_scores.sql
│           ├── gold_user_risk_profiles.sql
│           ├── gold_fraud_attribution.sql
│           ├── gold_model_evaluation.sql       # Precision/Recall/F1
│           └── gold_threshold_calibration.sql  # Threshold sensitivity
├── dashboard/
│   └── app.py                      # Streamlit dashboard (8 pages)
└── .github/workflows/
    └── pipeline.yml                # CI/CD automation
```

## Risk Scoring

The composite risk score (0–100) is a weighted sum of detected fraud signals:

| Signal | Weight | Rationale |
|---|---|---|
| Impossible Travel | 40 pts | Strongest signal — rarely a false positive at >500 mph |
| Velocity Spike | 35 pts | Card testing precursor — high urgency for blocking |
| Behavioral Drift | 25 pts | Spending anomalies — higher FP rate, requires human review |

**Risk Tiers**: CRITICAL (≥60) · HIGH (≥35) · MEDIUM (≥25) · LOW (<25)

All weights and tier cutoffs are externalized as dbt variables in `dbt_project.yml` and documented in `config.yml`, enabling data-driven recalibration without code changes.

## Model Evaluation

The pipeline computes full classification metrics against ground-truth labels:

| Metric | Description |
|---|---|
| **Precision** | TP / (TP + FP) — how many flags are real fraud |
| **Recall** | TP / (TP + FN) — how much fraud is caught |
| **F1-Score** | Harmonic mean of Precision and Recall |
| **False Positive Rate** | FP / (FP + TN) — operational noise level |
| **Confusion Matrix** | Full TP/FP/FN/TN breakdown per archetype |

The `gold_threshold_calibration` model evaluates detection performance across a range of thresholds per archetype, producing precision-recall tradeoff curves for optimal operating point selection.

## Calibration Methodology

1. **Initial weights** set by domain expertise (fraud analyst heuristics)
2. **Threshold sensitivity analysis** via `gold_threshold_calibration` — tests speed thresholds (200–800 mph), velocity counts (3–20 txns), and Z-Score bounds (1.5–5.0σ)
3. **Validation** against labeled ground truth using `gold_model_evaluation` — reports precision, recall, and F1 per archetype
4. **Production path**: weights would be tuned via logistic regression coefficients or gradient-boosted model feature importances; periodic recalibration (quarterly) as fraud patterns evolve

## Incremental Processing

Silver-layer models use dbt's `incremental` materialization strategy:
- On first run: full table build
- On subsequent runs: only processes transactions newer than `MAX(transaction_timestamp)` in the existing table
- Use `--full-refresh` flag to force a complete rebuild

This mirrors production patterns where millions of daily transactions are processed incrementally rather than rebuilt from scratch.

## Dashboard Pages

| Page | Purpose |
|---|---|
| **Overview** | Executive KPIs, fraud by archetype, timeline, accuracy |
| **Impossible Travel** | Speed distribution, geo map, flagged transactions |
| **Velocity Spikes** | Burst analysis, amount patterns |
| **Behavioral Drift** | Z-Score distribution, amount vs rolling average |
| **User Profiles** | Per-user risk aggregation, tier filtering |
| **Model Evaluation** | Precision/Recall/F1, confusion matrix, calibration curves |
| **Triage Queue** | Alert case management with disposition workflow |
| **Raw Explorer** | Ad-hoc SQL queries against DuckDB warehouse |

## Known Limitations & Future Work

- **Self-labeled data**: Fraud is injected by the same system that detects it, creating circular validation. A production system would use externally labeled data or blind holdout sets.
- **No temporal train/test split**: All evaluation is in-sample. Production deployment would require forward-looking validation on unseen future data.
- **Static thresholds**: Current thresholds are fixed. A production system would use adaptive thresholds that adjust to evolving fraud patterns (concept drift).
- **Scale ceiling**: DuckDB handles 1M+ records locally with ease. At 1B+ records, consider partitioned Parquet, incremental dbt models with date-based partitioning, and migration to cloud OLAP (Snowflake/BigQuery).
- **No real-time stream**: Current architecture is batch-oriented. Real-time fraud detection would require a streaming layer (Kafka + Flink) feeding into the same model logic.

