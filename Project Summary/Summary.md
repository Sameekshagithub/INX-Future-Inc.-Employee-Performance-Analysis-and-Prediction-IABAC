# Project Summary: Employee Performance Analysis, INX Future Inc.

## 1. Project summary

**What was done.** Data for 1,200 INX employees was cleaned, explored with statistical tests, and used to train and compare eight machine-learning algorithms that predict an employee's performance rating (2 Good, 3 Excellent, 4 Outstanding).

**Algorithm and training method.** The selected algorithm is a **Random Forest** (a large group of decision trees that vote), tuned by grid search with 5-fold stratified cross-validation on 80% of the data and evaluated once on the remaining 20% (240 employees). It reached **93.3% accuracy and a macro F1 of 0.90** on unseen employees (an always-predict-3 guess scores 72.9% and 0.28). Class weights were used because ratings are imbalanced.

**Most important features and why.** Selected by permutation importance on unseen data, cross-checked with built-in tree importance:
1. **Environment satisfaction**
2. **Last salary hike percentage**
3. **Time since last promotion**

Principal component analysis (PCA) was tested and deliberately **not** used, to keep results explainable.

**Other techniques and tools.** Python with pandas, NumPy, SciPy, scikit-learn, matplotlib and seaborn, in Jupyter notebooks; statistical tests (Kruskal-Wallis, chi-square, Spearman correlation, Cramér's V); engineered ratio features; decision-tree rules for plain-language indicators; out-of-fold predictions to build a fair watch list.

## 2. Features: selection and engineering

**Most important features selected and why.** The three factors above carry most of the signal: a model using only them reaches 86% accuracy and macro F1 0.81, about 90% of the full model's macro F1. They also have clear, different roles (see section 3). Personal characteristics (gender, marital status, education background, education level) showed no significant relationship with performance and carry almost no importance.

**Transformations made.**
- Three ratio features: promotion delay ratio, role stagnation ratio, average years per company. The promotion delay ratio replaced the raw years-since-promotion as the model's third most important factor; the two describe the same underlying fact.
- Binary encoding (gender, overtime, attrition), ordered encoding (travel frequency), one-hot encoding (education background, marital status, department, job role), and standardisation of numeric variables.

**Correlations and interactions.**
- Tenure variables (age, total experience, years at company, years in role, years with manager) are strongly correlated with each other (up to 0.78). This was handled by (a) using tree-based ensembles, which cope well with correlated inputs, and (b) judging importance through permutation importance on the whole model, and by grouping related variables when interpreting (for example, the two promotion variables are one factor).
- Interaction: the average rating is highest when environment satisfaction and salary hike are both high. Tree models capture such interactions automatically.

## 3. Results, analysis and insights

### 3.1 Interesting relationships

- **Environment satisfaction is a floor.** 97% of lower-rated (rating 2) employees report satisfaction 1 or 2. Nearly 4 in 10 employees with satisfaction 1-2 are rated 2, versus about 1 in 100 with satisfaction 3-4. Satisfaction 3 and 4 look identical, so the damage comes from *low* satisfaction, not from lacking *very high* satisfaction.
- **Salary hike marks the top.** Almost no one with a hike up to 17% is rated Outstanding, versus 61% of those with hikes above 21%. The hike is unrelated to who is rated 2.
- **Recent promotion protects.** About 1% of employees promoted this year are rated 2, versus 24-29% of those promoted earlier.
- **Fairness check.** Gender, marital status, education and travel show no significant relationship with rating. Overtime has a weak association in the *opposite* direction to the common assumption (11% of overtime workers are rated 2 versus 18% of others).
- **Department is not the root cause.** Development has *below-average* environment satisfaction yet the best ratings, so departmental differences are not explained by satisfaction alone; department and job role add only a small amount of predictive power once the three factors are known.

### 3.2 Most important technique

**Permutation importance on a held-out test set, combined with a tuned Random Forest.** It identifies which real-world factors the model depends on, on data it has not seen, and works on the original business variables rather than abstract components.

### 3.3 Answers to the business questions

**Insight 1: Department-wise performance**

| Department | Employees | Average rating | % rated 2 | % rated 3 | % rated 4 |
|---|---|---|---|---|---|
| Development | 361 | 3.09 | 3.6 | 84.2 | 12.2 |
| Data Science | 20 | 3.05 | 5.0 | 85.0 | 10.0 |
| Human Resources | 54 | 2.93 | 18.5 | 70.4 | 11.1 |
| Research & Development | 343 | 2.92 | 19.8 | 68.2 | 12.0 |
| Sales | 373 | 2.86 | 23.3 | 67.3 | 9.4 |
| Finance | 49 | 2.78 | 30.6 | 61.2 | 8.2 |

Differences are statistically significant (Kruskal-Wallis p < 0.001). **Development and Data Science perform best; Finance is weakest (almost one in three rated 2), followed by Sales and Research & Development.** Sales and Research & Development matter most in absolute terms: together they hold 155 of the 194 lower-rated employees. Data Science has only 20 employees, so its figures are less certain.

**Insight 2: Top 3 factors affecting performance**
1. Employee **environment satisfaction**
2. **Last salary hike percentage**
3. **Time since last promotion**

The first two are far ahead of all others (permutation importance about 0.27-0.30 versus 0.07 or less).

**Insight 3: Trained model.**
- *Current employees:* the Random Forest (`src/models/artifacts/performance_model.joblib`) predicts rating 2, 3 or 4 from the employee factors with about 93% accuracy on unseen employees; `predict_model.ipynb` shows how to use it for one or many employees.
- *Hiring:* a separate pre-hire model uses only candidate information. **It is barely better than chance** (balanced accuracy 0.37 vs 0.33 for chance) and must not be used for screening. This is a finding, not a failure: performance at INX is determined mostly by what happens *after* joining.

**Clear indicators of lower performers (the CEO's request).** Employees with **environment satisfaction of 1-2 and at least one year since last promotion** (23% of staff) include 96% of all rating-2 employees. A model-based watch list of 215 employees (18%) captures 90% of them with 81% precision.

### 3.4 Recommendations

*Principle: act on conditions, not on individuals. This addresses the CEO's morale concern, because it targets a minority and offers support rather than penalties.*

1. **Fix the working environment first, starting where it is weakest.** Roughly 38% of employees report environment satisfaction of 1-2, and this is the single biggest signal. Run short "stay interviews" and pulse surveys with this group (starting in Sales, Finance, Research & Development and Human Resources) to find *what* is driving low satisfaction, because the data records the score but not its causes. Track the share of employees at satisfaction 1-2 as a headline indicator.
2. **Close promotion gaps.** Introduce a mandatory career conversation for anyone more than a year without promotion or role change, especially in long-tenured roles; offer lateral moves, expanded responsibility or development plans where promotion is not available. Only about 1% of employees promoted this year are rated 2.
3. **Review how salary hikes are decided.** Hike size separates top performers sharply, which suggests hikes are linked to ratings. Check that this link is transparent and fair, and that employees with a small hike are told clearly what would earn a larger one. Do not assume that raising hikes alone will raise performance; the arrow may partly run from performance to hike, so trial any change first.
4. **Give extra support to the departments with the highest share of lower-rated staff**: Finance (31%), Sales (23%), Research & Development (20%), Human Resources (19%). Learn from Development, where only about 4% are rated 2.
5. **Use the watch list for support, never as a penalty list.** Share it only with the relevant manager and Human Resources. Every flagged person needs a conversation to find the cause first. Because the flag is built from workplace conditions (not gender, age or marital status) and covers a minority of staff, supportive action will not disturb the majority's morale. A rating of 2 is "Good", not a failure.
6. **Do not rely on hiring screens to fix performance.** The pre-hire model has almost no predictive power. Invest in onboarding, manager quality and retention conditions. Keep protected characteristics (gender, marital status) out of any hiring score.
7. **Close the data gaps** so the next analysis can answer what this one cannot: link employee data to client satisfaction and delivery metrics per team; keep performance ratings over time (several years); record manager identity and team workload; and add an open question to the satisfaction survey about *why* the score was given.
8. **Pilot, measure, repeat.** Start with one or two departments, compare against a control group, and re-run the model quarterly to track whether the share of employees at satisfaction 1-2 and rated 2 is falling.

### 3.5 More business insights

- **Satisfaction is a threshold effect.** Moving people from "low" to "high" matters; moving them from "high" to "very high" does not. The cheapest gains are in the low group.
- **Recent promotion is almost a protective factor.** Even a lateral or partial progression step may help; this is worth testing.
- **Top performers can be recognised through pay progression**, so retention of high performers depends on keeping hikes meaningful.
- **Performance is not about who people are.** Absence of any gender, marital-status or education effect means interventions can be organisation-wide in design rather than person-targeted.
- **The performance problem is concentrated.** About 16% of employees are rated 2, mostly in four departments and mostly among those reporting low environment satisfaction; the majority of employees (over 80%) are unaffected.

### 3.6 Limits to keep in mind

Associations are not proof of cause; the data is a single snapshot; there is no client-level data to link to the 8-point fall in client satisfaction; the smallest rating groups are small; and no employee has rating 1. Recommendations are therefore framed as things to pilot and measure.
