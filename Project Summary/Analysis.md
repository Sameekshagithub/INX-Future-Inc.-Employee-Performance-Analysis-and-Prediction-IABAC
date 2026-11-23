# Analysis: How the work was done and why

This document explains the choices made at each stage, from data preparation to the algorithm. The numbers come from the executed notebooks.

## 1. Data preparation (`data_processing.ipynb`)

| Check | Result |
|---|---|
| Missing values | None |
| Duplicate rows / duplicate employee numbers | None |
| Values outside the documented ranges | None |
| Logical consistency (years in role <= years at company <= total experience) | No violations |
| Outliers (beyond 1.5 times the interquartile range) | A few in tenure variables; genuine long-serving employees, so **kept** |

Because the data was clean, preparation focused on tidying text, adding readable rating labels, engineering features and encoding categories.

**Important observation:** no employee has rating 1; the lowest rating present is 2 (16% of employees). Class sizes: rating 2 = 194 (16.2%), rating 3 = 874 (72.8%), rating 4 = 132 (11.0%). The data is **imbalanced**, so a model that always answered "3" would be 73% accurate while being useless. Model quality is therefore judged mainly by **macro F1** (average F1 over the three ratings, with each rating counting equally).

## 2. Feature engineering

| New feature | Formula | Purpose |
|---|---|---|
| Promotion delay ratio | years since last promotion / (years at company + 1) | How much of an employee's time at INX has gone without promotion |
| Role stagnation ratio | years in current role / (years at company + 1) | Whether someone has stayed in one role |
| Average years per company | total experience / (companies worked + 1) | Stability versus job-hopping |

**Encoding.** Gender, overtime and attrition: 0/1. Business travel frequency: ordered (Non-Travel < Rarely < Frequently). Education background, marital status, department, job role: one-hot. Numeric variables are standardised inside the model pipeline. The employee number is excluded because it is only an identifier.

## 3. Exploratory analysis (`data_exploratory_analysis.ipynb`)

**Statistical tests used** (so conclusions do not rest on charts alone):
- Kruskal-Wallis and chi-square tests for department differences.
- Spearman rank correlation for numeric and ordinal factors versus rating.
- Chi-square test (with Cramér's V as effect size) for categorical factors.

**Findings**
1. Department differences are statistically significant (p < 0.001).
2. Strongest relationships with rating: environment satisfaction (rho = +0.40), last salary hike (+0.27), years since last promotion (-0.27); then tenure variables (negative).
3. Gender, marital status, education background, business travel and attrition: no significant relationship (p > 0.05).
4. The three factors play **different roles**:
   - Environment satisfaction acts as a **floor**: 188 of the 194 rating-2 employees (97%) report satisfaction 1 or 2, and of employees with satisfaction 1-2 about 41% are rated 2, versus about 1% for satisfaction 3-4. There is no difference between satisfaction 3 and 4.
   - Salary hike **separates the top performers**: with hikes up to 17%, almost nobody is rated 4; above 21%, 61% are rated 4. Hike size is unrelated to the share of rating-2 employees.
   - Recent promotion **protects against low ratings**: only about 1% of employees promoted this year are rated 2, versus 24-29% of everyone else.

**Multicollinearity.** Tenure variables are strongly correlated (for example, years at company with years in role: 0.76). This is a reason to prefer tree-based ensembles and permutation importance.

**Principal component analysis (PCA) was considered and not used.** 37 of 59 components were needed to keep 90% of the variance, so there is little compression to gain, and, more importantly, components blend many variables and cannot be explained to management. Interpretable features were kept.

## 4. Modelling (`train_model.ipynb`)

**Design.** 80/20 stratified train/test split (960 / 240). Model selection by 5-fold stratified cross-validation on the training part only. The test set was used once, at the end. Class weights were used to counter the imbalance.

**Eight algorithms compared** (cross-validated, before tuning):

| Algorithm | Accuracy | Macro F1 |
|---|---|---|
| Gradient Boosting | 0.925 | 0.885 |
| Random Forest | 0.922 | 0.879 |
| Decision Tree | 0.881 | 0.828 |
| Support Vector Machine | 0.804 | 0.733 |
| Neural Network | 0.823 | 0.719 |
| Logistic Regression | 0.760 | 0.695 |
| Extra Trees | 0.814 | 0.564 |
| K-Nearest Neighbours | 0.757 | 0.440 |

**Tuning.** A grid search over Random Forest and Gradient Boosting settings gave cross-validated macro F1 of 0.892 (Random Forest) and 0.890 (Gradient Boosting). These are practically tied; **Random Forest was selected** for its marginally higher score, stability and straightforward importance measures.

**Why tree-based ensembles win here.** The relationships are threshold-like and interactive (for example "satisfaction at most 2 AND no recent promotion"), which linear models and distance-based methods capture poorly.

**Final test-set result (240 unseen employees)**

| Measure | Random Forest | Always predict 3 |
|---|---|---|
| Accuracy | 0.933 | 0.729 |
| Macro F1 | 0.902 | 0.281 |
| Rating-2 employees found (recall) | 34 of 39 (87%) | 0 |
| Rating-4 employees found (recall) | 22 of 26 (85%) | 0 |

Errors are almost entirely between neighbouring ratings; no rating-2 employee was predicted as 4 on the test set. The test set is small, so figures carry uncertainty; cross-validation (macro F1 0.89) and out-of-fold predictions for all 1,200 employees (accuracy 0.92, macro F1 0.88) are consistent with it.

**Importance of factors.** Permutation importance on the test set (shuffling one original column at a time and measuring the drop in macro F1) was the main method, with built-in impurity importance as a cross-check:

| Rank | Factor | Permutation importance |
|---|---|---|
| 1 | Environment satisfaction | 0.295 |
| 2 | Last salary hike percentage | 0.268 |
| 3 | Time since last promotion (promotion delay ratio) | 0.065 |
| 4 | Years in current role | 0.044 |
| 5 | Job role | 0.027 |
| 6 | Department | 0.023 |

A model using only the top three factors reaches accuracy 0.86 and macro F1 0.81 (versus 0.93 / 0.90 for the full model).

**Pre-hire model.** The best predictors only exist after someone joins. A separate model was trained on candidate-knowable information (age, education, experience, number of past employers, role, level, pay, travel; gender and marital status deliberately excluded). On the test set it reached accuracy 0.50 and macro F1 0.33, with balanced accuracy 0.37 against 0.33 for chance, and about 4% precision on rating 4. **It is barely better than chance and cannot be used for screening.**

**Explainable indicator rules.** A shallow decision tree and a simple rule agree: flagging employees with **environment satisfaction of 1-2 and at least one year since last promotion** marks 281 employees (23% of staff), catching **96% of all rating-2 employees**; two thirds of those flagged are rated 2 (versus 16% overall).

**Watch list.** Using out-of-fold predictions, employees with at least 50% predicted probability of rating 2 form a list of 215 (17.9%) people; 81% of them are actually rated 2 and the list captures 90% of all rating-2 employees.

## 5. Limits of the analysis

1. **Correlation, not proof of cause.** Hikes and promotions are often given *for* good performance, and unhappy employees may be rated lower, so some arrows may run from rating to factor. Findings are strong indicators, to be confirmed with small trials.
2. **Snapshot data.** Nothing shows how performance changed over time.
3. **No client-level data**, so the link to the 8-point fall in client satisfaction cannot be tested.
4. **Small minority classes** (194 and 132 employees) limit precision of per-class estimates.
5. **Rating 1 does not occur**, so the model predicts only ratings 2, 3 and 4.
