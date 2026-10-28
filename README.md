# READ.md 
# Bank Marketing Direct Campaign Optimization & Term Deposit Prediction

**Team Name:** Team 27  
**Contributors:**  
- Leo Fajardo   
- Feodor Dronov 

---

## Objective and Motivation

### Dataset Details
- **Dataset Name:** Bank Marketing Dataset
- **Source:** UCI Machine Learning Repository
- **Direct Download Link:** [UCI Bank Marketing ZIP](https://archive.ics.uci.edu/static/public/222/bank+marketing.zip)
- **Collection Period:** May 2008 to November 2010
- **Domain & Context:** Direct phone-based telemarketing campaigns of a Portuguese retail banking institution.
- **Data Scale & Dimensions:** 45,211 instances (`bank-full.csv`), 16 input features, and 1 binary target feature (`y`).
- **Target Variable ($y$):** Has the client subscribed to a term deposit? (`"yes"` / `"no"`).
- **Key Features:**
  - *Demographics:* `age` (numeric), `job` (categorical: admin, blue-collar, entrepreneur, etc.), `marital` (categorical: married, single, divorced), `education` (categorical: primary, secondary, tertiary, unknown).
  - *Financial Profile:* `default` (credit in default: yes/no), `balance` (average yearly balance, in euros), `housing` (housing loan: yes/no), `loan` (personal loan: yes/no).
  - *Campaign Dynamics:* `contact` (communication type), `month`, `day`, `duration` (last contact duration in seconds; note: heavily predictive benchmark but must be handled carefully to avoid post-call leakage), `campaign` (number of contacts during campaign), `pdays`, `previous`, `poutcome` (outcome of previous campaign).
- **Data Hygiene State:** Semi-colon separated values (`;`). No explicit nulls, but frequent categorical `"unknown"` strings requiring encoding or imputation. Class distribution is heavily skewed/imbalanced (~11.7% `"yes"`, ~88.3% `"no"`).

### Motivation & Business Value
Financial institutions spend substantial resources operating outbound call centers. Cold-calling entire client rolls leads to high operational overhead and customer contact fatigue. By analyzing historical contact logs, we can isolate demographic and financial patterns that drive conversion. Building predictive classifications enables the bank to prioritize high-propensity leads, reducing total call volume while maintaining or boosting deposit acquisition rates.

### Core Questions, Expected Answers & Benefits

#### Research Question 1 (RQ1): Multivariate Profile Optimization
- **Question:** What combinations of demographic and operational characteristics (job, education, marital status, contact month) yield the highest conversion rates?
- **Expected Form of Answer:** Feature importance scores, decision tree split rules, and a ranked conversion rate matrix identifying top-performing customer segment combinations.
- **Benefit:** Allows marketing teams to segment the customer base and prioritize high-conversion sub-demographics over low-conversion profiles.

#### Research Question 2 (RQ2): Age Cohort Predictability & Variance
- **Question:** Which age brackets have the highest probability of subscription, and which bracket exhibits the highest variability (standard deviation) in conversion?
- **Expected Form of Answer:** Mean conversion probability and standard deviation $\sigma$ across binned age cohorts (e.g., `<25`, `25–39`, `40–54`, `55–64`, `65+`), supported by ANOVA tests.
- **Benefit:** Resolves whether young clients are high-variance/speculative leads while senior clients offer lower-volume but more predictable acceptance rates.

#### Research Question 3 (RQ3): Existing Debt Burden vs. Propensity to Invest
- **Question:** Are individuals with existing credit liabilities (`housing` loan, personal `loan`, or both) significantly less likely to open a term deposit than debt-free clients?
- **Expected Form of Answer:** Odds ratios (OR) derived from logistic regression and Chi-Square ($\chi^2$) tests of independence comparing conversion proportions across four debt permutations.
- **Benefit:** Informs whether marketing should automatically exclude clients holding active debt commitments from term deposit campaigns.

#### Research Question 4 (RQ4): Liquid Account Balance Thresholds
- **Question:** What is the quantitative relationship between a customer's liquid average yearly account `balance` and their probability of opening a term deposit?
- **Expected Form of Answer:** Empirical CDF (Cumulative Distribution Function), logistic probability curves, and quartile conversion tables isolating critical balance tipping points.
- **Benefit:** Establishes minimum balance thresholds for campaign targeting, preventing wasted outreach on capital-constrained accounts.

---

## Introduction & Context of Project
Direct marketing campaigns in retail banking face diminishing returns under mass-outreach models. The UCI Bank Marketing dataset presents a rich intersection of demographic markers, financial solvency metrics, and historical contact touchpoints collected across Portuguese banking operations during the 2008–2010 financial climate. 

Understanding how structural features—such as debt burden, liquid savings, and age cohorts—govern an individual's decision to lock capital into a term deposit provides actionable leverage. Through exploratory data analysis, hypothesis testing, and multivariate classification, this project identifies optimal lead scoring rules to drive efficiency in customer acquisition.

---

## Implementation Details

### Data Manipulation & Pipeline Strategy
1. **Parsing & Ingestion:** Read semi-colon delimited file (`sep=';'`) using `pandas`.
2. **Missing & Unknown Data:** Map categorical `"unknown"` tokens to explicit NaN or evaluate them as discrete categorical levels where domain-relevant (e.g., `poutcome="unknown"` denotes a first-time lead).
3. **Imbalance Mitigation:** Implement Stratified K-Fold cross-validation, class-weight balancing (`class_weight='balanced'`), or SMOTE oversampling to prevent bias toward the majority class (`y="no"`).
4. **Encoding & Scaling:** Apply One-Hot Encoding (`pd.get_dummies` / `OneHotEncoder`) to categorical predictors (`job`, `marital`, `education`, `loan`, `housing`). Apply robust scaling (`StandardScaler` or `RobustScaler`) to skewed continuous variables (`balance`, `duration`, `age`).

### Methods, Metrics, Tools, and Plots Matrix

| Question | Metrics & Statistical Values | Python Toolboxes & Functions | Effective Plots & Visualizations |
| :--- | :--- | :--- | :--- |
| **RQ1: Multivariate Drivers** | Gini Importance, Permutation Importance, F1-score, Precision-Recall AUC | `sklearn.ensemble.RandomForestClassifier`, `sklearn.tree.DecisionTreeClassifier`, `sklearn.metrics` | Feature Importance horizontal bar chart, decision tree path visualization, and multi-factor conversion heatmaps (`seaborn.heatmap`) |
| **RQ2: Age Distributions** | Mean conversion ($\mu$), Standard Deviation ($\sigma$), Interquartile Range (IQR), One-way ANOVA $F$-statistic | `pandas.cut` (binning), `numpy`, `scipy.stats.f_oneway` | Binned Age Bar Chart with standard deviation error bars, faceted KDE distribution overlays (`seaborn.histplot(kde=True)`) |
| **RQ3: Loan Impact** | Pearson Chi-Square ($\chi^2$), $p$-value, Odds Ratio (OR), Conversion Percentage | `scipy.stats.chi2_contingency`, `statsmodels.api.Logit` | Grouped Bar Chart comparing subscription rates across debt profiles: [None, Housing Only, Personal Only, Both] (`seaborn.barplot`) |
| **RQ4: Balance Correlation** | Point Biserial Correlation ($r_{pb}$), Mann-Whitney $U$ test, Median Balance differences | `scipy.stats.pointbiserialr`, `scipy.stats.mannwhitneyu`, `statsmodels.nonparametric` | Log-transformed Violin/Box Plot of balance vs. subscription status; empirical sigmoid logistic probability curve |

### Implementation Artifacts (Phase-2 Preview)
*System architecture diagrams, correlation matrices, and model pipelines will be documented here.*

```python
# [Placeholder] Top-Level Pipeline Orchestration Architecture
def run_bank_marketing_pipeline(data_path: str):
    """
    1. Load semi-colon delimited CSV.
    2. Execute demographic and balance transformations.
    3. Run Chi-Square and ANOVA hypothesis tests.
    4. Fit classification baseline and output diagnostic plots.
    """
    pass
