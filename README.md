# 🏦 Loan Default Risk Intelligence System

> A full end-to-end Business Analyst portfolio project covering SQL, Python, statistical testing, and predictive modelling on 265,000+ real loan records.

---

## 📌 Project Summary

Retail lending institutions face a growing Non-Performing Loan (NPL) problem. This project simulates a real BA engagement: starting from a business problem statement, moving through data discovery, statistical validation, A/B testing, and delivering an operational risk framework a credit team can use on day one.

| Item                      | Detail                                                       |
| ------------------------- | ------------------------------------------------------------ |
| **Dataset**               | Lending Club Loan Data (2007–2018)                           |
| **Rows analysed**         | 265,776 resolved loans                                       |
| **Target variable**       | Default (Charged Off = 1, Fully Paid = 0)                    |
| **Baseline default rate** | ~20%                                                         |
| **Tools**                 | Python · MySQL · Scikit-learn · Scipy · Matplotlib · Seaborn |
| **Project duration**      | 6 phases across 5 notebooks                                  |

---

## 🎯 Business Problem

> _"Our NPL ratio has risen from 4.2% to 6.8% in 24 months — well above the 3.5% industry benchmark. Manual review queues are up 40% and cost per loan decision has increased. We have no systematic way to identify which borrowers carry disproportionate default risk before approval."_

**Project goal:** Build a data-driven risk tier framework that reduces NPL ratio to ≤5.0% within 2 quarters of adoption, without requiring new systems or replacing the existing credit scoring model.

---

## 📁 Repository Structure

```
loan-default-risk-intelligence/
│
├── notebooks/
│   ├── 01_data_loading.ipynb       # Raw data ingestion, cleaning, type casting
│   ├── 02_mysql_upload.ipynb       # Push clean data to MySQL via SQLAlchemy
│   ├── 03_eda.ipynb                # Exploratory analysis, 6 charts, feature engineering
│   ├── 04_ab_testing.ipynb         # 3 hypothesis tests, power analysis, A/B test
│   └── 05_model.ipynb              # Logistic regression, ROC, confusion matrix, risk tiers
│
├── sql/
│   └── profiling_queries.sql       # 8 SQL queries for data profiling (MySQL syntax)
│
├── outputs/
│   ├── cleaned_loans.csv           # Cleaned dataset (post-filtering)
│   ├── featured_loans.csv          # Feature-engineered dataset
│   ├── loan_predictions.csv        # Model output with risk tier per loan
│   ├── ab_test_summary.csv         # A/B test results summary
│   ├── model_card.json             # Model performance card (AUC, recall, F1)
│   └── charts/                     # All 11 output charts (PNG, 150 DPI)
│
└── README.md
```

---

## 🔬 Phase-by-Phase Breakdown

### Phase 1 — Problem Framing

- Wrote a Business Requirements Document (BRD) with stakeholder map and sign-off matrix
- Defined 8 KPIs across three tiers: lagging (NPL ratio, AUC), leading (manual review rate), and guardrail (fair lending compliance)
- Mapped 6 stakeholders: CRO, Underwriting, Compliance, IT, Finance, Borrowers

### Phase 2 — Data Discovery & SQL

- Built a 22-column data dictionary with business meaning and data quality notes
- Wrote 8 profiling queries in MySQL covering: baseline audit, null analysis, default by grade, DTI bands, income × verification cross-tab, FICO distribution, delinquency risk matrix, and quarterly cohort trend
- Key data quality issues resolved: `loan_status` filtering, `int_rate` string casting, `revol_util` outlier capping, class imbalance documentation

### Phase 3 — EDA & Feature Engineering (Python)

- Produced 6 charts: grade default rate, DTI bands, FICO distribution, quarterly trend, correlation heatmap, FICO × delinquency risk matrix
- Engineered 5 new features:

| Feature             | Logic                           | Why                                          |
| ------------------- | ------------------------------- | -------------------------------------------- |
| `payment_to_income` | installment / (annual_inc / 12) | Captures affordability better than DTI alone |
| `credit_util_ratio` | open_acc / total_acc            | Measures credit line saturation              |
| `log_annual_inc`    | log1p(annual_inc)               | Fixes right-skew for modelling               |
| `fico_band`         | pd.cut on fico_mid              | Categorical risk tier from FICO score        |
| `has_delinquency`   | delinq_2yrs > 0                 | Binary signal outperforms raw count          |

### Phase 4 — Statistical Analysis & A/B Testing

**3 Hypothesis Tests:**

| Test                    | Method         | Result                                                    |
| ----------------------- | -------------- | --------------------------------------------------------- |
| Grade vs. Default       | Chi-square     | p ≈ 0 · REJECT H0 · Grade is significant predictor        |
| FICO vs. Default        | Welch's t-test | p ≈ 0 · ~20–25pt gap · Cohen's d = medium                 |
| Delinquency vs. Default | Chi-square     | p < 0.05 · REJECT H0 · Delinquency history is significant |

**Power Analysis:** Required ~8,000 samples per group. Dataset has 265K — sufficient with high margin.

**A/B Test:** Simulated risk-tiered approval policy (treatment: flag DTI>25 + delinquency + Grade E-G) vs. current policy (control). Two-proportion z-test confirmed statistically significant default rate reduction at α=0.05.

### Phase 5 — Predictive Model

**Model:** Logistic Regression with StandardScaler pipeline

| Metric        | Value                           | KPI Target | Status  |
| ------------- | ------------------------------- | ---------- | ------- |
| Test AUC      | [0.736]                         | ≥ 0.75     | [✓ / ✗] |
| 5-fold CV AUC | [0.732 0.737 0.739 0.735 0.734] | ≥ 0.72     | [✓ / ✗] |
| Recall        | [0.672]                         | ≥ 0.70     | [✓ / ✗] |
| Precision     | [0.338]                         | —          | —       |

**Key design decisions:**

- `class_weight='balanced'` — handles 80/20 class imbalance
- `Pipeline(scaler + model)` — prevents data leakage between train and test sets
- `stratify=y` in train/test split — preserves class distribution
- `drop_first=True` in one-hot encoding — avoids dummy variable trap

**Top 5 default risk drivers (by coefficient magnitude):**

1. `int_rate` — higher interest rate → higher default probability
2. `fico_mid` — higher FICO → lower default probability
3. `dti` — higher debt-to-income → higher default probability
4. `has_delinquency` — any past delinquency → higher default probability
5. `grade_B/C/D...` — lower grades → progressively higher default probability

**Risk Tier Output:**

| Tier     | Default Probability | Action             | Volume            |
| -------- | ------------------- | ------------------ | ----------------- |
| 🟢 Green | < 15%               | Auto-approve       | ~[11.9]% of loans |
| 🟡 Amber | 15–30%              | Manual review      | ~[22.5]% of loans |
| 🔴 Red   | > 30%               | Decline / escalate | ~[53.9]% of loans |

---

## 📊 Key Findings

1. **Grade predicts default 5×** — Grade A ~6% vs Grade G ~32% default rate. Statistically confirmed (χ²=3000+, p≈0).
2. **DTI is an underused signal** — Very High DTI (30+) borrowers default at 2× the rate of Low DTI (<10) borrowers.
3. **FICO gap is real and meaningful** — ~20–25 point FICO difference between defaulters and payers, confirmed by t-test with medium Cohen's d.
4. **Delinquency compounds risk** — Past delinquency + High revolving utilisation is the worst-performing segment (28–35% default rate).
5. **NPL is trending upward** — Quarterly cohort analysis confirms rising default rates in 2015–2018 cohorts, coinciding with volume growth.
6. **Risk-tiered policy works** — A/B test shows statistically significant default rate reduction under the proposed policy (p < 0.05).

---

## 🛠️ Tech Stack

```
Language  : Python 3.11 (Anaconda)
Notebook  : Jupyter Notebook
Database  : MySQL 8.x via MySQL Workbench
ORM       : SQLAlchemy + PyMySQL
Libraries : pandas · numpy · matplotlib · seaborn
            scipy · scikit-learn
IDE       : Jupyter + MySQL Workbench
OS        : Windows 11
```

---

## ⚙️ Setup Instructions

### 1. Clone the repo

```bash
git clone https://github.com/Devvrat12/loan-default-risk-intelligence.git
cd loan-default-risk-intelligence
```

### 2. Install dependencies

```bash
pip install pandas numpy matplotlib seaborn scikit-learn scipy pymysql sqlalchemy jupyter
```

### 3. Download the dataset

Download `accepted_2007_to_2018Q4.csv.gz` from:
[Kaggle — Lending Club Loan Data](https://www.kaggle.com/datasets/wordsforthewise/lending-club)

Place it in: `data/accepted_2007_to_2018Q4.csv.gz`

### 4. Set up MySQL

```sql
CREATE DATABASE loan_risk;
```

Update the connection string in `02_mysql_upload.ipynb`:

```python
engine = create_engine('mysql+pymysql://root:yourpassword@localhost:3306/loan_risk')
```

### 5. Run notebooks in order

```
01 → 02 → 03 → 04 → 05
```

> **Note:** Shut down each notebook after running before opening the next — frees RAM on machines with <8GB available.

---

## 📋 Deliverables Checklist

- [x] Business Requirements Document (BRD)
- [x] Stakeholder map with power/interest grid
- [x] KPI framework (lagging / leading / guardrail)
- [x] Data dictionary (22 columns with business meaning)
- [x] 8 SQL profiling queries (MySQL)
- [x] 6 EDA charts (PNG, 150 DPI)
- [x] 5 engineered features with documented rationale
- [x] 3 hypothesis tests with effect sizes
- [x] Power analysis and A/B test design
- [x] Logistic regression model with Pipeline
- [x] ROC curve, confusion matrix, feature importance chart
- [x] Green / Amber / Red risk tier framework
- [x] Model card (JSON)
- [x] Business impact estimate ($)

---

## 🙋 About

**Devvrat Tiwari** — Final-year B.Tech CSE (AI/ML), JUET Guna  
AI/ML Developer Intern @ Assemble (Esports, Bharatpur)

📎 [LinkedIn](https://linkedin.com/in/devvrattiwari) · [GitHub](https://github.com/Devvrat12)

> _"The hardest part isn't the model — it's translating a business problem into a testable hypothesis, and a model output into an operational decision framework."_

---

_Dataset source: Lending Club public loan data via Kaggle. Used for educational and portfolio purposes only._
