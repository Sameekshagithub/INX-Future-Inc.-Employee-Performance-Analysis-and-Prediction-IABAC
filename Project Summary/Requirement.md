# Requirement: What the project asks for

## 1. The business situation (plain-language explanation)

**INX Future Inc.** (INX) is a data analytics and automation company with over 15 years of global presence. It has been rated a top-20 employer for five years in a row, and its people policies are seen as employee friendly.

**The problem.** In recent years employee performance has become unhealthy. Escalations on service delivery have increased, and client satisfaction has fallen by 8 percentage points.

**The constraint.** The Chief Executive Officer, Mr. Brain, does not want to simply penalise weak performers. Penalties could hurt the morale of *all* employees (and further reduce performance) and could damage INX's reputation as a best employer, which helps it attract talent.

**What he asked for.** A data science project that analyses current employee data to find the **underlying causes** of the performance problem, and gives **clear indicators of lower-performing employees**, so that any action can be targeted without disturbing everyone else.

## 2. The four required insights

| # | Required insight | Where it is answered |
|---|---|---|
| 1 | Department-wise performance | `src/data_processing/data_exploratory_analysis.ipynb`, Figure 2, Summary section 4.1 |
| 2 | Top 3 important factors affecting performance | `src/models/train_model.ipynb` (section 5), Figure 4, Summary section 4.2 |
| 3 | A trained model that predicts performance from input factors (to be used for hiring) | `src/models/train_model.ipynb`, `predict_model.ipynb`, saved in `src/models/artifacts/`, Summary section 4.3 |
| 4 | Recommendations to improve performance | Summary section 4.4 |

## 3. The data

One workbook with two sheets (stored unchanged in `data/raw/`):

- **Employee records**: 1,200 employees and 28 columns. Each row is one employee.
- **Data definitions**: meaning of the coded values (for example, satisfaction 1 = Low ... 4 = Very High).

| Group | Columns |
|---|---|
| Identity and personal | Employee number, age, gender, education background, education level, marital status |
| Job | Department, job role, job level, hourly rate, business travel frequency, distance from home, overtime |
| Experience and career | Total work experience, number of companies worked, years at this company, years in current role, years since last promotion, years with current manager, training times last year |
| Satisfaction scores (1-4) | Environment satisfaction, job satisfaction, job involvement, relationship satisfaction, work-life balance |
| Pay | Hourly rate, last salary hike percentage |
| Outcomes | Attrition (left the company: yes/no) and **performance rating** (the target) |

**The target variable, performance rating** takes values 2 (Good), 3 (Excellent) or 4 (Outstanding). The definitions also list 1 (Low), but **no employee has that rating**. In this project, "lower-performing" therefore means rating 2.

## 4. What IABAC (the certifying body) requires in the submission

From the submission guidelines:

- A project summary: algorithms and training methods, important features and why, other techniques and tools.
- Feature selection and engineering: important features, transformations, correlations and interactions.
- Results, analysis and insights: interesting relationships, the most important technique, clear answers to the business problems, further business insights.
- Work delivered as **Jupyter notebooks** with explanatory text, in the **prescribed directory structure**, with titles that avoid abbreviations.
- Submission within 7 days of receiving the project, at the address given in the project brief.

All of these are covered by this project folder (see `README.md` for the map).

## 5. Things the project brief does not provide

- **Client satisfaction and service delivery data.** The brief mentions an 8-point drop in client satisfaction, but there is no client data in the file, so this project cannot directly link employee factors to client outcomes.
- **Time dimension.** The data is a single snapshot. It cannot show *how* performance changed over the past years.
- **Reasons behind low environment satisfaction.** The score is recorded, but not its causes.

These limits are reflected in the recommendations.
