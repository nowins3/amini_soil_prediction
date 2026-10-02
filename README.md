# Amini Soil Prediction

An exploratory machine learning notebook for the Amini soil prediction competition on Zindi. It predicts 11 soil nutrients from the supplied features, compares several regression models on a shared validation split, and converts test predictions into nutrient gaps for a submission CSV.

The notebook is a reproducible **implementation**, not a record of verified competition scores. Earlier experiments were removed from the original notebook, so their settings and results are not reconstructed here.

## What is included

- Feature engineering for NDVI and EVI, based on the original notebook.
- Missing-value imputation fitted on training data, with numeric scaling for the linear models and categorical encoding where needed.
- RidgeCV (L2), LassoCV (L1), and ElasticNetCV (L1/L2), tuned separately for each nutrient with cross-validation.
- Random Forest with controls on leaf size and feature sampling.
- Optional XGBoost and LightGBM candidates with regularization and sampling settings.
- Per-nutrient RMSE and MAE, a model comparison table, and a nitrogen residual plot.
- A final refit on all labeled rows and generation of `submission.csv` in the format `ID,Gap`.

## Project layout

```text
.
├── README.md
├── upload/
│   └── amini_soil_prediction.ipynb
└── data/                         # Add your competition CSVs here; do not commit them without permission
    ├── Train.csv
    ├── Test.csv
    ├── Gap_Test.csv
    └── VariableDefinitions.csv   # Optional
```

The notebook is currently at [`upload/amini_soil_prediction.ipynb`](upload/amini_soil_prediction.ipynb). If you move it to the repository root, adjust this link; its default data directory remains `data/` relative to the directory where you run Jupyter.

## Setup

Use Python 3 with Jupyter Notebook or JupyterLab. In the environment used by the notebook, install:

```bash
python -m pip install jupyter numpy pandas matplotlib seaborn scikit-learn
```

For the optional gradient boosting models, also install:

```bash
python -m pip install xgboost lightgbm
```

Place `Train.csv`, `Test.csv`, and `Gap_Test.csv` in `data/`, then open and run the notebook from top to bottom. Alternatively, set `AMINI_DATA_DIR` to the directory containing those files **before** starting the notebook kernel:

```bash
export AMINI_DATA_DIR=/path/to/amini-data
jupyter lab
```

On Windows PowerShell, set the variable with `$env:AMINI_DATA_DIR = 'C:\path\to\amini-data'`. `VariableDefinitions.csv` is used for a display table if present; it is not required to train or generate a submission. No Google Drive mount, API token, or private local path is required by the notebook.

## Data and modeling

The notebook expects the competition CSV column names. `Train.csv` needs `PID`, the nutrient targets `N`, `P`, `K`, `Ca`, `Mg`, `S`, `Fe`, `Mn`, `Zn`, `Cu`, and `B`, plus spectral bands `mb1`, `mb2`, and `mb3`. `Test.csv` needs `PID`, those spectral bands, and `BulkDensity`. `Gap_Test.csv` needs `PID`, `Nutrient`, and `Required`. Other shared feature columns are used as predictors except for `site` and `PID`.

The notebook creates NDVI and EVI and drops the bands `mb1`, `mb2`, `mb3`, `mb7`, and `parv` from predictors, following the original feature engineering. It then uses an 80/20 random holdout for the model comparison. The regularized linear estimators select penalties using cross-validation **within the training partition**; the holdout is not passed to those fits.

By default it compares RidgeCV, LassoCV, ElasticNetCV, and Random Forest. To include XGBoost and LightGBM, install them and set `RUN_OPTIONAL_BOOSTERS = True` in the candidate-model cell. The lowest mean nutrient RMSE becomes `SELECTED_MODEL` by default; edit `SELECTED_MODEL` if a different choice fits your evaluation goal. Reported holdout scores are local diagnostics and have not been verified against the competition scoring system. A random split can overstate performance when training and test locations are related.

## Submission

After selection, the notebook refits the chosen pipeline on all labeled data and predicts the nutrient values for `Test.csv`. It uses the original notebook's conversion with a **20 cm assumed soil depth**:

```text
available_kg_ha = predicted_ppm × 20 × BulkDensity × 0.1
Gap = Required − available_kg_ha
ID = PID + "_" + Nutrient
```

The final cell writes `submission.csv` in the current working directory with columns `ID` and `Gap`, in the order of `Gap_Test.csv`. Confirm the competition's current submission rules and conversion assumptions before uploading. The competition CSVs were not supplied with this repository, so no scores or predictions on the real data are included.
