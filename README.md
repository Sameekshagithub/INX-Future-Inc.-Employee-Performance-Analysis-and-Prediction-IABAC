# Employee Performance Analysis: INX Future Inc.

Certified Data Scientist project (project code 10281) for the International Association of Business Analytics Certification (IABAC).

**Start with `Project Summary/Summary.md`.** It contains the answers to the four required insights.

## How to run

```bash
pip install -r requirements.txt
# then run the notebooks in this order (each one saves outputs for the next):
#   src/data_processing/data_processing.ipynb
#   src/data_processing/data_exploratory_analysis.ipynb
#   src/models/train_model.ipynb       (takes a few minutes)
#   src/models/predict_model.ipynb
#   src/visualization/visualize.ipynb
```
All notebooks are already executed, so outputs can be read without re-running.

## Structure (as required by the submission guidelines)

```
INX_Future_Employee_Performance_Project
├── Project Summary
│   ├── Requirement.md   <- what the project asks for, in plain language
│   ├── Analysis.md      <- choices made: data preparation, features, algorithm, results
│   └── Summary.md       <- summary, answers, insights and recommendations
├── data
│   ├── external         <- (none used)
│   ├── processed        <- cleaned, model-ready data and the watch list
│   └── raw              <- the original, unmodified workbook
├── src
│   ├── data_processing
│   │   ├── data_processing.ipynb
│   │   └── data_exploratory_analysis.ipynb
│   ├── models
│   │   ├── train_model.ipynb
│   │   ├── predict_model.ipynb
│   │   └── artifacts     <- trained models, metrics, importances, predictions
│   └── visualization
│       ├── visualize.ipynb
│       └── figures       <- the nine result figures (PNG)
└── references           <- data definitions and the original project documents
```

## Headline results
- Random Forest: 93.3% accuracy, macro F1 0.90 on 240 unseen employees.
- Top 3 factors: environment satisfaction, last salary hike percentage, time since last promotion.
- Weakest departments: Finance, Sales, Research & Development. Strongest: Development, Data Science.
- A model using only candidate information (for hiring) is barely better than chance.
