# Fake Job Posting Detection

<<<<<<< HEAD
Data science project (Part A). Authors: [add your names here]
=======
Data science project(Part-A)

ME24B1016-Suranjana Mary, ME24B1020-Sushmitha, ME24B1025-Rakshana M
>>>>>>> 0aa1447369f8a06468691afb5ee5e687a79bbe06

## Problem

Online job scams put job seekers at risk of fraud and identity theft. This project builds a model that flags fraudulent job postings from their text and metadata (company profile, logo, screening questions, contact details, description and so on).

Only about 5% of postings are fake, so accuracy is misleading: a model that always says "real" scores about 95%. We judge models on the **fake class** using precision, recall, F1 and PR-AUC. Wrongly blocking a genuine employer is costly, so we look at precision and recall together instead of recall alone.

## Data

[Real / Fake Job Posting Prediction (EMSCAD)](https://www.kaggle.com/datasets/shivamb/real-or-fake-fake-jobposting-prediction) on Kaggle.

- 17,880 postings and 18 columns as downloaded, with `fraudulent` (0 = real, 1 = fake) as the target.
- After removing 281 exact duplicate postings: 17,599 rows, of which 856 are fake (4.86%).

The data file is **not included** in this repository. Download `fake_job_postings.csv` from Kaggle and put it in `data/`.

## Approach

**`notebooks/01_eda.ipynb`: cleaning and exploration**
- Removed duplicates, filled missing values with constants and built flags such as `has_company_profile`, `has_salary`, `has_email` and `has_phone`.
- Found that a missing company profile, no logo, no screening questions and an email or phone number in the text all raise the fake rate. A missing profile gives about 17.7% fake vs 1.9% when present.
- Cleaned the text (masked links, emails and phone numbers replaced with single words, HTML entities decoded).
- Made five plots with written insights and saved the cleaned data to `data/jobs_clean.csv`.

**`notebooks/02_modelling.ipynb`: models and evaluation**
- Stratified split: 60% train, 20% validation, 20% test.
- All preprocessing (TF-IDF, scaling) sits inside scikit-learn pipelines fitted on the training rows only.
- Features: 7 binary flags, top-5 country dummies, description length and word count, and TF-IDF (2,000 unigram and bigram features) on the cleaned text.
- Only the models taught in the course are used:
  1. Linear regression (score turned into a class with a threshold)
  2. Logistic regression (class-weight balanced, `C` tuned on validation PR-AUC)
  3. Logarithmic regression (linear regression on `log1p` of the length features)
- The decision threshold for each model is chosen on the validation set to maximise F1. The best model is chosen by validation PR-AUC. The test set is used once, at the end.

## Results (test set: 3,520 postings, 171 fake)

| Model | Precision | Recall | F1 | PR-AUC |
|---|---|---|---|---|
| Linear regression | 0.806 | 0.632 | 0.708 | 0.793 |
| Logistic regression (tuned) | 0.692 | 0.813 | 0.747 | 0.841 |
| Logarithmic regression | 0.844 | 0.632 | 0.722 | 0.792 |

The final model is logistic regression. It has the highest PR-AUC (and ROC-AUC, 0.981) and catches about 139 of the 171 fake postings, at the cost of about 62 false alarms among 3,349 real postings. Linear and logarithmic regression perform almost the same, because the TF-IDF features dominate and the log transform affects only two columns.

With only 171 fake postings in the test set, the recall estimate is uncertain by roughly 6 percentage points, and F1 differences of about 0.01 between models are not meaningful.

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

- `random_state=42` is used for the splits and the models.
- The notebooks run top to bottom without manual fixes.
- Results can differ slightly with other library versions, so `requirements.txt` pins the versions we used.

## Limitations

- About one in five fake postings is missed at the chosen threshold, and about one in three flagged postings is actually real. Catching more fakes means more false alarms.
- A single length term cannot capture the U-shaped relationship between description length and fake rate.
- The `country` feature may reflect how the data was collected, not how scams work.
- Near-duplicate and template postings may remain after removing exact duplicates.
<<<<<<< HEAD
- Professional-looking scams are missed, and the text model uses word counts only.

Possible next steps: sentence embeddings or a transformer model, a human review step for postings near the cut-off, and testing on newer data.
=======
- The text model uses word counts only, so professional-looking scams can be missed.
>>>>>>> 0aa1447369f8a06468691afb5ee5e687a79bbe06
