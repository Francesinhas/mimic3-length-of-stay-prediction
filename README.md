# MIMIC-III ICU Length of Stay Prediction

> Predicting ICU Length of Stay for Pneumonia Patients Using Clinical Time-Series Data

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![MIMIC-III](https://img.shields.io/badge/Dataset-MIMIC--III-green.svg)](https://mimic.physionet.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Authors

- **Cosmin-Nicolae Tianu**
- **Codruta-Elen Jucan**
- **George Tofan**

---

## Table of Contents

- [Overview](#overview)
- [Motivation](#motivation)
- [Dataset](#dataset)
- [Project Structure](#project-structure)
- [Methodology](#methodology)
- [Feature Engineering](#feature-engineering)
- [Models & Results](#models--results)
- [Installation](#installation)
- [Usage](#usage)
- [Notebooks](#notebooks)
- [Data Files](#data-files)
- [Challenges & Limitations](#challenges--limitations)
- [Future Work](#future-work)
- [References](#references)

---

## Overview

This project develops a machine learning pipeline to **predict the Length of Stay (LOS)** for patients admitted to the Intensive Care Unit (ICU) with **bacterial pneumonia** (ICD-9 code: 48283) using the **MIMIC-III** (Medical Information Mart for Intensive Care) clinical database.

Accurate LOS prediction is critical for:
- **Resource allocation** and ICU bed management
- **Clinical decision-making** and discharge planning
- **Cost estimation** for hospital administration
- **Improving patient outcomes** through early intervention

## Motivation

### Clinical Significance

ICU Length of Stay prediction remains a challenging yet essential task in critical care medicine. Early and accurate predictions enable:

1. **Operational efficiency**: Better bed management and staffing allocation
2. **Patient care**: Tailored treatment plans based on expected recovery trajectories
3. **Cost management**: More accurate billing and resource utilization forecasting

### Why Pneumonia?

We selected **bacterial pneumonia (ICD-9: 48283)** as our target condition based on:

- **High ICU admission rates**: Pneumonia is a leading cause of ICU admissions
- **Sufficient sample size**: 145 ICU stays with comprehensive chart event data
- **Clinical relevance**: Well-defined clinical markers for monitoring
- **Temporal predictability**: Clear patterns within the first 24 hours of ICU admission

---

## Dataset

### MIMIC-III Database

The [MIMIC-III database](https://mimic.physionet.org/) is a freely accessible critical care database containing de-identified health data from ~40,000 patients admitted to Beth Israel Deaconess Medical Center ICUs between 2001-2012.

### Tables Used

| Table | Description | Key Fields |
|-------|-------------|------------|
| `PATIENTS` | Patient demographics | `SUBJECT_ID`, `GENDER`, `DOB` |
| `ADMISSIONS` | Hospital admission records | `HADM_ID`, `ADMITTIME`, `ETHNICITY` |
| `ICUSTAYS` | ICU stay information | `ICUSTAY_ID`, `INTIME`, `LOS` |
| `CHARTEVENTS` | Clinical measurements | `ITEMID`, `VALUENUM`, `CHARTTIME` |
| `DIAGNOSES_ICD` | ICD-9 diagnosis codes | `ICD9_CODE` |
| `D_ITEMS` | Item/measurement descriptions | `ITEMID`, `LABEL` |

### Cohort Statistics

| Metric | Value |
|--------|-------|
| Hospital admissions (pneumonia) | 264 |
| ICU stays with chart data | 145 |
| Chart events analyzed | ~987,000 |
| Unique clinical items | 474 |
| LOS range | 1.1 - 40+ days |
| Median LOS | ~5.5 days |

### Key Clinical Measurements (Top 20)

| Rank | Item | Unit | Frequency |
|------|------|------|-----------|
| 1 | Heart Rate | bpm | 4,498 |
| 2 | Respiratory Rate | insp/min | 4,446 |
| 3 | O2 Saturation (SpO2) | % | 4,444 |
| 4 | Arterial BP (Mean) | mmHg | 2,759 |
| 5 | Arterial BP (Systolic) | mmHg | 2,738 |
| 6 | Arterial BP (Diastolic) | mmHg | 2,738 |
| 7 | Inspired O2 Fraction (FiO2) | - | 2,090 |
| 8 | PEEP Set | cmH2O | 1,955 |
| 9 | Mean Airway Pressure | cmH2O | 1,929 |
| 10 | Tidal Volume (observed) | mL | 1,923 |

---

## Project Structure

```
MIMIC-III-Predicting-LOS/
│
├── README.md                   # Project documentation
├── Report.pdf                  # Detailed project report
├── plan.md                     # Development planning notes
│
├── notebooks/
│   ├── report.ipynb                    # Main analysis pipeline
│   ├── Interpretation.ipynb            # Model interpretation & SHAP analysis
│   ├── ExploratoryDataAnalysis.ipynb   # Initial data exploration
│   ├── FeatureEngineering.ipynb        # Feature extraction pipeline
│   ├── InitialPrediction.ipynb         # Baseline model experiments
│   ├── DiseaseChoice.ipynb             # Disease selection analysis
│   ├── Queries.ipynb                   # SQL queries for data extraction
│   ├── 24hVisualisation.ipynb          # Time-series visualization
│   └── pipeline_stages_documentation.ipynb
│
└── data_sets/
    ├── d_pneumonia.csv              # Chart events for pneumonia cohort (~987K rows)
    ├── d_bronchiectasis.csv         # Chart events for bronchiectasis cohort
    ├── d_systolic_heart.csv         # Chart events for systolic heart failure
    ├── los_pneumonia.csv            # LOS target variable (145 ICU stays)
    ├── norm_pneumonia.csv           # Normalized time-series data (~92K rows)
    ├── items_freq_pneumonia.csv     # Item frequency statistics
    └── items_appearance_pneumonia.csv  # Item appearance across patients
```

---

## Methodology

### 1. Data Preprocessing Pipeline

```
Raw MIMIC-III Tables
        ↓
┌───────────────────────────────────────────┐
│  1. Disease Selection (ICD-9: 48283)      │
│  2. Table Merging (Patients + ICU Stays)  │
│  3. Outlier Exclusion                     │
│     - LOS < 1 day (removed)               │
│     - Extreme LOS outliers (capped)       │
│  4. Age Harmonization                     │
│     - Patients >89 → set to 91.4          │
│  5. First 24-Hour Window Extraction       │
└───────────────────────────────────────────┘
        ↓
   Preprocessed Cohort
```

### 2. Feature Engineering

#### Static Features
- **Demographics**: Age, Gender (label encoded)
- **Ethnicity**: One-hot encoded (grouped rare categories as "OTHER")
- **Hospital Time**: Duration between hospital and ICU admission

#### Time-Series Aggregation (First 24 Hours)

For each clinical measurement (ITEMID), we computed:

| Aggregation | Description |
|-------------|-------------|
| `mean` | Average value over 24 hours |
| `std` | Standard deviation (variability) |
| `count` | Number of measurements |
| `range` | Max - Min (spread) |
| `trend` | Linear slope (trajectory) |

#### Target Transformation

- **Log-transformation** applied to LOS to address right-skew
- Results in more Gaussian-like distribution for better model performance

### 3. Feature Selection

Three-stage feature selection pipeline:

1. **Variance Threshold** (threshold=0.01)
   - Remove near-constant features
   
2. **Correlation Filtering** (threshold=0.9)
   - Remove highly correlated feature pairs
   - Reduces multicollinearity

3. **Permutation Importance** (threshold=1%)
   - XGBoost-based feature importance
   - Retain only predictive features

### 4. Model Training

#### Algorithm: XGBoost Regressor

Hyperparameter tuning via `GridSearchCV`:

```python
params = {
    'max_depth': [3, 5, 7, 10],
    'learning_rate': [0.01, 0.1],
    'n_estimators': [100, 200],
    'reg_alpha': [0.1]
}
```

#### Cross-Validation Strategy

- **GroupKFold** (n_splits=5) to prevent patient leakage
- Groups defined by `SUBJECT_ID`

---

## Models & Results

### Performance Metrics

| Metric | Value |
|--------|-------|
| Mean Absolute Error (MAE) | ~5.2 days |
| Root Mean Squared Error (RMSE) | ~7.0 days |
| R-squared (R²) | ~0.35 |

### Key Predictive Features (SHAP Analysis)

Top features identified through SHAP values:

1. **Mean Platelet Count** - Strong indicator of illness severity
2. **Mean Heart Rate** - Vital sign stability
3. **Heart Rate Alarm (High) Trend** - Clinical deterioration signal
4. **Respiratory Rate Statistics** - Pulmonary function
5. **O2 Saturation Variability** - Oxygenation stability

### Model Interpretation

SHAP (SHapley Additive exPlanations) was used for:
- Global feature importance ranking
- Local prediction explanations
- Understanding feature interactions

---

## Installation

### Prerequisites

- Python 3.8+
- Access to MIMIC-III database (requires credentialed access via [PhysioNet](https://physionet.org/))

### Dependencies

```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost shap jupyter
```

Or install all requirements:

```bash
pip install -r requirements.txt
```

### Required Libraries

```python
# Data Processing
pandas>=1.3.0
numpy>=1.21.0

# Visualization
matplotlib>=3.4.0
seaborn>=0.11.0

# Machine Learning
scikit-learn>=0.24.0
xgboost>=1.4.0

# Interpretability
shap>=0.39.0

# Statistical Analysis
statsmodels>=0.12.0
```

---

## Usage

### 1. Data Preparation

If you have access to MIMIC-III, extract the required tables:

```sql
-- Example: Extract pneumonia chart events
SELECT ce.* 
FROM CHARTEVENTS ce
JOIN DIAGNOSES_ICD d ON ce.HADM_ID = d.HADM_ID
WHERE d.ICD9_CODE = '48283'
```

### 2. Run the Pipeline

```bash
# Start Jupyter
jupyter notebook

# Open notebooks/report.ipynb for the complete pipeline
```

### 3. Key Steps

1. **Load Data**: Import preprocessed CSV files
2. **Preprocess**: Handle missing values, outliers, age encoding
3. **Feature Engineering**: Aggregate time-series, encode categoricals
4. **Feature Selection**: Apply variance, correlation, and importance filters
5. **Train Model**: XGBoost with cross-validation
6. **Evaluate**: Compute MAE, RMSE, R²
7. **Interpret**: Generate SHAP plots

---

## Notebooks

| Notebook | Purpose |
|----------|---------|
| `report.ipynb` | **Main pipeline** - Complete end-to-end analysis |
| `Interpretation.ipynb` | SHAP analysis and model interpretation |
| `ExploratoryDataAnalysis.ipynb` | Initial data exploration and statistics |
| `FeatureEngineering.ipynb` | Detailed feature extraction experiments |
| `InitialPrediction.ipynb` | Baseline model experiments |
| `DiseaseChoice.ipynb` | Analysis for selecting target disease |
| `Queries.ipynb` | SQL queries for MIMIC-III data extraction |
| `24hVisualisation.ipynb` | Time-series visualization of ICU data |

---

## Data Files

### Processed Datasets

| File | Rows | Description |
|------|------|-------------|
| `d_pneumonia.csv` | 986,963 | Chart events for pneumonia cohort |
| `norm_pneumonia.csv` | 92,008 | Normalized first-24h time-series |
| `los_pneumonia.csv` | 145 | LOS target values per ICU stay |
| `items_freq_pneumonia.csv` | 474 | Clinical item frequency statistics |
| `items_appearance_pneumonia.csv` | 474 | Item appearance across patients |

### Alternative Disease Cohorts

| File | Rows | Disease |
|------|------|---------|
| `d_bronchiectasis.csv` | 201,245 | Bronchiectasis |
| `d_systolic_heart.csv` | 182,483 | Systolic Heart Failure |

---

## Challenges & Limitations

### Data Challenges

1. **Skewed LOS Distribution**
   - Most stays ≤22 days with long tail of extended stays
   - Addressed via log-transformation

2. **Sparse, Irregular Time-Series**
   - Clinical measurements at varying frequencies
   - Handled via aggregation (mean, std, trend)

3. **High Dimensionality**
   - Thousands of potential predictors
   - Multi-stage feature selection required

4. **Small Sample Size**
   - Only 145 ICU stays with complete chart data
   - Limited model complexity

### Methodological Limitations

- **Single disease focus**: Results may not generalize to other conditions
- **24-hour window**: Excludes later clinical developments
- **Aggregation loss**: Time-series patterns reduced to summary statistics
- **MIMIC-III age**: Data from 2001-2012 may not reflect current practices

---

## Future Work

1. **Deep Learning Models**
   - LSTM/Transformer for raw time-series
   - Attention mechanisms for feature importance

2. **Multi-Disease Modeling**
   - Transfer learning across conditions
   - Condition-specific prediction heads

3. **Extended Temporal Windows**
   - 48-hour and 72-hour prediction windows
   - Dynamic predictions updated over time

4. **Additional Data Sources**
   - Laboratory values (LABEVENTS)
   - Medication data (PRESCRIPTIONS)
   - Clinical notes (NOTEEVENTS via NLP)

5. **Clinical Validation**
   - Prospective validation on MIMIC-IV
   - External validation on eICU dataset

---

## References

1. Johnson, A.E.W., et al. (2016). MIMIC-III, a freely accessible critical care database. *Scientific Data*, 3:160035.

2. Harutyunyan, H., et al. (2019). Multitask learning and benchmarking with clinical time series data. *Scientific Data*, 6:96.

3. Lundberg, S.M. & Lee, S.I. (2017). A unified approach to interpreting model predictions. *NeurIPS*.

4. PhysioNet: https://physionet.org/content/mimiciii/

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- MIMIC-III Database: MIT Laboratory for Computational Physiology
- PhysioNet for providing open access to clinical data
- Beth Israel Deaconess Medical Center for the original data collection

---

<p align="center">
  <i>For questions or collaboration, please open an issue or contact the authors.</i>
</p>