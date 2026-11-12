# Fake Job Posting Detection

Data science project (Part A). Authors: [add your names here]

## Problem

Online job scams put job seekers at risk of fraud and identity theft. This project builds a model that flags fraudulent job postings from their text and metadata (company profile, logo, screening questions, contact details, description and so on).

Only about 5% of postings are fake, so accuracy is misleading: a model that always says "real" scores about 95%. We judge models on the **fake class** using precision, recall, F1 and PR-AUC. Our goal is to catch at least 75% of fake postings while keeping false alarms low, because wrongly blocking a genuine employer is costly.

## Data

[Real / Fake Job Posting Prediction (EMSCAD)](https://www.kaggle.com/datasets/shivamb/real-or-fake-fake-jobposting-prediction) on Kaggle.

- 17,880 postings and 18 columns as downloaded, with `fraudulent` (0 = real, 1 = fake) as the target.
- After removing 281 exact duplicate postings (no label conflicts): 17,599 rows, of which 856 are fake (4.86%).
- Postings are from 2012 to 2014.

The data file is **not included** in this repository. Download `fake_job_postings.csv` from Kaggle and put it in `data/`.

## Approach

**`notebooks/01_eda.ipynb`: cleaning and exploration**
- Removed duplicates, filled missing values with constants and built flags such as `has_company_profile`, `has_salary`, `has_email` and `has_phone`.
- Found that a missing company profile, no logo, no screening questions and an email or phone number in the text all raise the fake rate.
- Replaced masked links, emails and phone numbers in the text with single words, decoded HTML entities, and verified the replacement against independent counts.
- Made five plots with written insights (saved in `reports/figures/`) and saved the cleaned data to `data/jobs_clean.csv`.

**`notebooks/02_modelling.ipynb`: models and evaluation**
- Stratified 80/20 train/test split. 5-fold cross-validation on the training set is the validation step.
- All preprocessing (TF-IDF, scaling, encoding) sits inside scikit-learn pipelines, so nothing is learned from validation or test rows.
- Models: dummy baseline, logistic regression, linear SVM and random forest. The two linear models were tuned with grid search scored on PR-AUC.
- The decision threshold for each model was chosen on out-of-fold training scores to reach 75% recall. The test set was used once, at the end.
- Error analysis of missed fakes, false alarms and model coefficients.

## Results (test set: 3,520 postings, 171 fake)

| Model | Precision | Recall | F1 | PR-AUC |
|---|---|---|---|---|
| Logistic regression (tuned) | 0.963 | 0.754 | 0.846 | 0.934 |
| Linear SVM (tuned) | 0.962 | 0.749 | 0.842 | 0.940 |
| Random forest | 0.859 | 0.749 | 0.800 | 0.908 |

The final model is the tuned linear SVM, chosen before the test set was opened. It caught 128 of 171 fake postings and raised 5 false alarms on 3,349 real postings. The logistic regression performs just as well. The random forest is weaker, with lower precision at the same recall.

With only 171 fake postings in the test set, the recall estimate is uncertain by roughly 6 percentage points.

## How to run

Requires Python 3.10 or newer.

```bash
# 1. Create and activate a virtual environment
python -m venv venv
venv\Scripts\activate          # Windows
source venv/bin/activate       # Mac/Linux

# 2. Install the libraries
pip install -r requirements.txt

# 3. Put fake_job_postings.csv in data/

# 4. Run the notebooks in order (Run All in each)
#    notebooks/01_eda.ipynb        -> creates data/jobs_clean.csv and the figures
#    notebooks/02_modelling.ipynb  -> trains, tunes and evaluates the models
```

Open the notebooks in VS Code or Jupyter and select the `venv` kernel. `02_modelling.ipynb` needs the file created by `01_eda.ipynb`, so run them in that order.

## Project structure

```
fake-job-detection/
├── data/
│   ├── fake_job_postings.csv   (download from Kaggle)
│   └── jobs_clean.csv          (created by 01_eda)
├── notebooks/
│   ├── 01_eda.ipynb
│   └── 02_modelling.ipynb
├── reports/
│   └── figures/                (plots saved by the notebooks)
├── requirements.txt
└── README.md
```

## Reproducibility

- `random_state=42` is used for the split, the cross-validation folds and every model.
- The notebooks run top to bottom without manual fixes (checked with Restart and Run All).
- Results can differ slightly with other library versions, so `requirements.txt` pins the versions we used.

## Limitations

- Small test set (171 fakes), so recall is uncertain by about 6 points.
- About a quarter of fake postings are missed at the chosen setting. Catching more means many more false alarms.
- The data is from 2012 to 2014 and scam patterns have changed.
- The `country` feature may reflect how the data was collected, not how scams work.
- Near-duplicate and template postings may remain after removing exact duplicates.
- Professional-looking scams are missed, and the text model uses word counts only.

Possible next steps: sentence embeddings or a transformer model, a human review step for postings near the cut-off, and testing on newer data.
