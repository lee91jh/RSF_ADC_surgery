# Random survival forest for overall survival after resection of lung adenocarcinoma

Code accompanying the manuscript:

> [Development and validation of a random survival forest model for postoperative survival in lung adenocarcinoma]. [Jong Hyuk Lee, Yelee Kwon, Jae Kwang Yun, Geun Dong Lee, Hae Won Lee, Chang Min Choi, Hyeong Ryul Kim]. [Surgical Oncology], [Year]. doi: [DOI]

The repository contains the core modeling pipeline used in the study: preprocessing, hyperparameter search and
training of a random survival forest (RSF), performance evaluation, permutation-based feature importance, and
predicted survival curves for simulated patient profiles.

## Data availability

The patient-level registry data used in this study cannot be shared publicly because of patient privacy and
institutional regulations. The trained model is not distributed for the same reason.

`data/dummy_data.csv` is a **synthetic dataset** with the same variable names and coding as the study data
(6,000 rows; train/validation/test = 4,200/900/900). Values were randomly permuted across rows and columns, and the
survival outcome is unrelated to the predictors. The dummy data allow every notebook to run end to end, but
**the results do not reproduce the values reported in the manuscript** (expect C-index ≈ 0.5 on the validation
and test sets).

To apply the pipeline to your own data, provide a CSV with the same columns and set `DATA_PATH` in the first code
cell of each notebook.

## Repository structure

```
.
├── README.md
├── LICENSE
├── requirements.txt
├── data/
│   └── dummy_data.csv                      synthetic example data
├── notebooks/
│   ├── 01_data_preprocessor.ipynb
│   ├── 02_RSF_train.ipynb
│   ├── 03_RSF_test_performance.ipynb
│   ├── 04_RSF_permutation_importance.ipynb
│   └── 05_simulated_patients.ipynb
└── outputs/                                fitted preprocessor and model (created when running, not tracked)
```

| Notebook | Content | Manuscript |
|---|---|---|
| `01_data_preprocessor` | Builds the preprocessing pipeline and fits it on the training set | 
| `02_RSF_train` | Optuna hyperparameter search (optional) and training of the final RSF |
| `03_RSF_test_performance` | C-index, IBS, time-dependent ROC/AUC, point-specific metrics, calibration |
| `04_RSF_permutation_importance` | Grouped and individual permutation importance of the RSF |
| `05_simulated_patients` | Predicted survival curves for simulated patient profiles |

Each notebook is self-contained; shared settings (paths, data loading, metrics) are defined in its first code cells.

## How to run

```bash
pip install -r requirements.txt
cd notebooks
jupyter notebook
```

Run `01` and `02` first; they write `outputs/preprocessor.joblib` and `outputs/RSF_model.joblib`, which `03`–`05`
load. Notebooks `03`–`05` are independent of each other.

* `02_RSF_train`: `RUN_OPTUNA = False` (default) trains the final model with the selected hyperparameters;
  set `RUN_OPTUNA = True` to repeat the 200-trial search.
* `04_RSF_permutation_importance`: 100 repetitions per feature or feature group (`N_REP`). This can take several
  hours; reduce `N_REP` for a quick test run.

Tested with Python 3.12 and the package versions in `requirements.txt`.

## Methods summary

**Preprocessing** (`01_data_preprocessor`). Numeric variables: median imputation and min-max scaling.
Biomarkers: "not performed" (-1) treated as missing and imputed with the mode. Symptoms: binary indicators.
Categorical variables: one-hot encoding. The pipeline is fitted on the training set only. Variable coding is given
in Supplementary Table 1 of the manuscript.

**Model.** `sksurv.ensemble.RandomSurvivalForest` with 100 trees, `min_samples_split = 10`,
`min_samples_leaf = 5`, `max_features = "log2"`, `random_state = 42`, selected by maximizing the validation
C-index over 200 Optuna trials.

**Performance metrics.**
* C-index: Harrell's concordance index.
* Integrated Brier score: 100-point time grid from the minimum to the maximum follow-up of the evaluated set
  (end point excluded); censoring distribution estimated from the training set.
* Time-dependent AUC at 2, 5, 7 and 10 years: cumulative/dynamic AUC with inverse probability of censoring
  weighting (Uno); score = predicted probability of death by *t*, 1 − S(*t*); censoring distribution from the
  training set.
* Point-specific accuracy, F1 score, precision and recall: deaths before *t* vs. patients censored at or after
  *t*; threshold = Youden index on the validation set.

**Permutation importance.** Each predictor or predefined clinical block is shuffled in the test set 100 times;
the mean C-index decrease and IBS increase with 95% intervals (2.5th–97.5th percentile) are reported.

## Citation

If you use this code, please cite the article above.

## License

Released under the MIT License (see `LICENSE`).
