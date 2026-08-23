# Vehicle Insurance Fraud Analysis

An end-to-end analysis of vehicle insurance claims to identify fraud patterns and risk factors, combining SQL analysis, Python-based data cleaning and modeling, and a Power BI dashboard.

## Problem Statement

Insurance fraud costs the industry billions annually, but only a small fraction of claims are actually fraudulent (~6% in this dataset) — making it a classic imbalanced classification problem. This project identifies which claim characteristics are most associated with fraud, and builds a model to flag high-risk claims for review.

## Tools Used

- **SQL (SQLite)** — data querying and aggregation
- **Python (pandas, scikit-learn)** — data cleaning, exploratory analysis, and fraud classification modeling
- **Power BI** — interactive dashboard for stakeholder-facing insights

## Dataset

[Vehicle Insurance Claim Fraud Detection](https://www.kaggle.com/datasets/shivamb/vehicle-claim-fraud-detection) (Kaggle) — 15,420 claims, 33 features, originally an Oracle real-world fraud case study.

## Data Quality Audit

Before analysis, the raw data was audited and found to have several real-world data quality issues, not just a "clean" Kaggle export:

- **Hidden missing values**: `'0'` used as a placeholder in `DayOfWeekClaimed`/`MonthClaimed` instead of a true null
- **Placeholder ages**: `Age = 0` for some records, imputed using the midpoint of the corresponding `AgeOfPolicyHolder` bin
- **Typos in vehicle make**: "Nisson," "Mecedes," "Porche" corrected to standard spellings
- **Ordinal data stored as text ranges**: columns like `PastNumberOfClaims`, `AgeOfVehicle`, `NumberOfSuppliments` were encoded numerically to preserve their natural order rather than one-hot encoded
- **No duplicate rows** found
- **Significant class imbalance**: 94% not-fraud vs 6% fraud, requiring precision/recall/AUC evaluation instead of accuracy

## SQL Analysis — Key Findings

Queries available in [`sql/fraud_analysis_queries.sql`](sql/fraud_analysis_queries.sql).

| Finding | Detail |
|---|---|
| **Fault is the strongest signal** | Policy Holder at fault → 7.89% fraud rate vs. Third Party at fault → 0.88% — a 9x difference |
| **Coverage type matters** | Liability-only claims have near-zero fraud (0.72%); All Perils/Collision claims (which pay out on the policyholder's own vehicle) run 10-14% |
| **Vehicle make varies** | Accura, Saturn, Saab show the highest fraud rates (10-12.5%), though some of these have small sample sizes |
| **Mild seasonality** | March and August see the highest fraud rates (~7.5%), November the lowest (3.83%) |
| **Counterintuitive**: fraud rate *decreases* with more past claims (7.79% for 0 past claims → 3.38% for 3+) — worth further investigation rather than an intuitive "repeat offender" pattern |

## Model

A Random Forest classifier was trained to predict fraud, using `class_weight='balanced'` to address the class imbalance.

| Metric | Logistic Regression | Random Forest |
|---|---|---|
| ROC-AUC | 0.80 | **0.82** |
| Fraud Recall | 0.92 | 0.70 |
| Fraud Precision | 0.13 | 0.16 |

**Random Forest was selected** as the better-balanced model. It catches 70% of fraud cases at the cost of a higher false-positive rate — an acceptable and common tradeoff in fraud detection, where missing real fraud is costlier than a false alarm.

**Feature importance confirmed the SQL findings**: `Fault` (21.5%) and `BasePolicy`/`PolicyType` (14.4% + 7.8%) were the top predictors — the same drivers identified independently through SQL analysis, lending confidence to both approaches.

## Dashboard

The Power BI dashboard (`Insurance_Fraud_Dashboard.pbix`) presents:
- KPI summary: total claims, total fraud cases, overall fraud rate
- Fraud rate by fault, vehicle make, policy type, and month

![Dashboard Screenshot](dashboard_screenshot.png)

## Business Recommendation

Given that **fault status and coverage type** are the strongest fraud predictors, claims where the policyholder is at fault under All Perils/Collision coverage warrant additional review, while Liability-only claims could see a streamlined approval process given their near-zero fraud rate.

## Project Structure

```
├── sql/
│   └── fraud_analysis_queries.sql
├── notebooks/
│   └── fraud_analysis.ipynb
├── data/
│   ├── fraud_oracle.csv (raw)
│   └── claims_cleaned.csv (cleaned)
├── powerbi/
│   └── Insurance_Fraud_Dashboard.pbix
├── dashboard_screenshot.png
└── README.md
```

## Note on Scope

The core deliverable of this project is SQL analysis and the Power BI dashboard — data analyst scope. A basic classification model was added to validate the SQL findings and generate risk scores, extending into light predictive modeling to demonstrate analytical range beyond descriptive reporting.
