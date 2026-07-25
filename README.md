# Consumer Loan Charge-Off Risk Analysis

This is an end-to-end credit risk analytics project making use of LendingClub consumer-loan data, DuckDB, Python, logistic regression, and Tableau. This project analyzes historical loan outcomes, then builds an interpretable model using information available at loan application, and then evaluates whether the model can help identify loans that may require additional review.

## Business question

Which loanee/loan characteristics are associated with charge-offs, and how effectively can a model identify loans with higher charge-off risk?

## Project overview

The original LendingClub file contains 2M+ accepted loans from 2007–2018. The pipeline was:

1. Loaded and queried the raw CSV directly with DuckDB.
2. Restricted the outcome to terminal statuses: `Fully Paid`, `Charged Off`, and `Default`.
3. Checked whether loans from each origination year had enough time to reach a final outcome, preventing active    loans from being incorrectly treated as fully repaid.
4. Used mature 2011–2013 originations for modeling.
5. Trained on 2011–2012 loans and tested on the later 2013 loans.
6. Restricted model inputs to information available when the loan originated.
7. Built an interpretable logistic regression model and evaluated its ranking performance.
8. Created a Tableau dashboard combining portfolio findings and model results.

## Dashboard

**[View the interactive dashboard on Tableau Public](https://public.tableau.com/app/profile/meet.shah1034/viz/ConsumerLoanCharge-OffRiskAnalysis/Dashboard1)**

The packaged Tableau workbook is also available at [`outputs/final.twbx`](outputs/final.twbx).

The dashboard contains two complementary types of analysis.

### Portfolio analysis

- Total loans, originated principal, and overall charge-off rate
- Observed charge-off rate by LendingClub grade
- Observed charge-off rate by loan term

### Model analysis

- Observed charge-off rate by predicted-risk decile
- Cumulative share of charge-offs captured by the highest-risk loans
- Permutation feature importance

## Data and cohort design

Only loans with known  outcomes were used. Loans labeled `Current`, late, or in a grace period were not used because their final repayment outcomes were unknown.

Later years were excluded from modeling because many loans issued near the end of the dataset did not have enough time to reach a known outcome. Restricting the model to 2011–2013 reduced the risk of retaining mostly fast-resolving charge-offs from later years while excluding loans that would eventually be fully paid.

The mature modeling population contained **209,892 loans** and approximately **$2.96 billion** in originated principal. The held-out 2013 test portfolio contained:

- **134,804 loans**
- **$1.98 billion** in originated principal
- **15.6%** observed charge-off rate

## Leakage controls

The model excludes variables that would only be observed after the loan was given, including payments received, recoveries, outstanding principal, and last-payment information.

It also excludes the original data's own `grade`, `sub_grade`, and `int_rate`. These variables are used for descriptive portfolio analysis, but not using them in the model prevents the logistic regression from simply reproducing LendingClub's existing risk assessment.

## Modeling approach

- **Model:** Logistic regression
- **Training period:** 2011–2012
- **Test period:** 2013
- **Numeric preprocessing:** Median imputation and standardization
- **Categorical preprocessing:** Most-frequent imputation and one-hot encoding
- **Primary metrics:** ROC-AUC, average precision, risk-decile performance, and cumulative charge-off capture
- **Interpretation:** Permutation importance on the held-out test cohort

The project prioritizes interpretability and sound validation over extensive model tuning. The model produces a continuous risk ranking; a real lending organization would select an operational review threshold based on expected credit losses, review capacity, customer impact, fairness, and regulatory requirements.

## Results

- **ROC-AUC:** 0.673
- **Average precision:** 0.267, compared with a 0.156 no-skill baseline
- The highest-risk **10%** of loans contained **20.9%** of all observed charge-offs.
- The highest-risk **20%** contained **37.0%** of charge-offs.
- The highest-risk **30%** contained **50.1%** of charge-offs.
- The highest-risk decile had a **32.7%** charge-off rate, compared with **4.8%** in the lowest-risk decile.
- Loan term, FICO score, annual income, loan amount, and recent credit inquiries were the most important model features.

The portfolio analysis also showed that 60-month loans had materially higher observed charge-off rates than 36-month loans. This finding is consistent with loan term ranking as the model's strongest feature.

## Repository structure

```text
lendingclub-risk-analysis/
├── data/                         # Raw LendingClub CSV (not tracked)
├── notebooks/
│   ├── 01_exploration.ipynb     # Cleaning, cohort audit, and SQL portfolio analysis
│   ├── 02_modeling.ipynb        # Streamlined modeling and export workflow
│   └── 02_modeling_full.ipynb   # Detailed modeling backup
├── outputs/
│   ├── final.twbx               # Packaged Tableau dashboard
│   ├── feature_importance.csv
│   ├── model_metrics.csv
│   └── risk_decile_summary.csv
├── .gitignore
└── README.md
```

## Tools

- **DuckDB / SQL:** Large-file ingestion, filtering, aggregation, and portfolio analysis
- **Python:** Data preparation and export
- **pandas / scikit-learn:** Preprocessing, logistic regression, evaluation, and permutation importance
- **Tableau:** Interactive portfolio and model-performance dashboard

## Reproducing the analysis

1. Download the LendingClub accepted-loans 2007–2018 dataset.
2. Place `accepted_2007_to_2018Q4.csv` inside `data/`.
3. Run `notebooks/01_exploration.ipynb`.
4. Run `notebooks/02_modeling.ipynb`.
5. Open `outputs/final.twbx` in Tableau Public or Tableau Desktop.

The raw dataset and local DuckDB database are intentionally excluded from version control because of their size.

## Limitations

- LendingClub borrowers and underwriting practices may not represent other lenders or current credit conditions.
- The analysis is observational and does not establish that a feature causes charge-off.
- The logistic model provides moderate discrimination and is not intended for production lending decisions.
- Production use would require calibration, stability monitoring, fairness testing, economic threshold analysis, etc.
