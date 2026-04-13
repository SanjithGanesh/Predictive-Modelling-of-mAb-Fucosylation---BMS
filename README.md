# 🧬 Predictive Modelling of mAb Fucosylation in CHO Cell Bioprocessing

> **MSc Data Science — BMS Capstone Project**
> Benchmarking 7 ML models to predict monoclonal antibody (mAb) fucosylation from CHO cell culture conditions, with automated data cleansing and an interactive results dashboard.

---

## 📌 Project Overview

Fucosylation is a critical quality attribute (CQA) in monoclonal antibody manufacturing. This project builds a complete ML pipeline to **predict fucosylation levels** from bioprocess parameters (temperature, pH, dissolved oxygen, nutrient concentrations, etc.) using CHO (Chinese Hamster Ovary) cell culture data.

The core experiment investigates a fundamental question in bioprocessing ML: **How does dataset size affect model performance across different model architectures?**

### Key Findings

| Metric | BMS1 (N=500) | BMS2 (N=10,000) |
|--------|:------------:|:----------------:|
| **Best Model** | GPR (R²=0.75) | XGBoost+SHAP (R²=0.83) |
| **ANN Performance** | R²=0.05 ❌ | R²=0.77 ✅ (+1440%) |
| **Physics Features** | Captured 64.7% of full performance | Captured 74.7% of full performance |
| **Recommended for Production** | GPR | XGBoost+SHAP |

---

## 📂 Repository Structure

```
├── BMS/                          # Experiment 1 — Small Dataset (N=500)
│   ├── data/                     # Generated synthetic dataset (500 samples, 10 batches)
│   ├── models/                   # Trained model artifacts
│   │   ├── ridge/
│   │   ├── plsr/
│   │   ├── random_forest/
│   │   ├── ann/
│   │   ├── gpr/
│   │   ├── hybrid/
│   │   └── xgboost_shap/
│   └── results/                  # Performance metrics, plots, SHAP outputs
│
├── BMS2/                         # Experiment 2 — Large Dataset (N=10,000)
│   ├── data/                     # Generated synthetic dataset (10,000 samples, 50 batches)
│   ├── models/                   # Trained model artifacts (same 7 architectures)
│   └── results/                  # Performance metrics, plots, comparison outputs
│
├── automation/                   # Automated Data Cleansing Module
│   ├── cleaner.py                # Core cleansing pipeline
│   ├── preprocessor.py           # Feature engineering & transformation
│   └── pipeline_runner.py        # End-to-end: ingest → clean → run 7 models → export results
│
├── dashboard/                    # Interactive Results Dashboard
│   ├── app.py                    # Dashboard application
│   ├── visualizations/           # Chart components & templates
│   └── assets/                   # Static assets for dashboard UI
│
├── README.md
└── requirements.txt
```

> **Note:** Adjust folder/file names above to match your actual repo structure.

---

## 🔬 Models Benchmarked

### 1. Ridge Regression (Linear Baseline)
- Full coefficient interpretability
- BMS1: R²=0.51 → BMS2: R²=0.57
- Structural ceiling — cannot capture nonlinear relationships

### 2. Partial Least Squares Regression (PLSR)
- Identified 3 significant features via VIP scores: GDP-Fucose (1.92), Temperature (1.76), Mn (1.20)
- BMS1: R²=0.51 → BMS2: R²=0.57
- Same linear limitation as Ridge

### 3. Random Forest
- Captured nonlinear effects (pH quadratic, temperature saturation)
- MDI and permutation importance consistency (Spearman ρ=0.96)
- BMS1: R²=0.66 → BMS2: R²=0.79 (+20%)

### 4. Artificial Neural Network (ANN)
- **Headline finding:** R²=0.05 → R²=0.77 (+1440% improvement)
- At N=500: 12,225 parameters vs 340 training samples = massively overparameterised
- At N=10,000: 6,800 training samples finally sufficient for convergence
- LIME explanations validate feature attributions at N=10,000

### 5. Gaussian Process Regression (GPR)
- Best performer at N=500 with R²=0.75
- RBF kernel; calibration mean error 0.83%; 95% CI coverage exactly 95.0%
- Limited improvement at N=10,000 (R²=0.79) due to O(n³) scaling (300-sample subsample cap)

### 6. Hybrid Model (Physics-Informed)
- 5 biologically engineered features: FPI (substrate×cofactor), Energy Charge, Golgi Proxy, Stress Index, Enzyme Activity
- Engineered features alone captured **64.7% (BMS1) → 74.7% (BMS2)** of full model performance with 50% fewer variables
- Demonstrates domain knowledge value for dimensionality reduction

### 7. XGBoost + SHAP
- **Best production model** — top accuracy with full explainability
- BMS1: R²=0.74 → BMS2: R²=0.83
- SHAP feature rankings match published biochemistry
- Detected GDP-Fucose × Mn synergy interaction consistent with ground truth

---

## 📊 Performance Comparison

### BMS1 (N=500)

| Model | R² | RMSE | Key Strength |
|-------|:--:|:----:|-------------|
| Ridge | 0.51 | — | Interpretability |
| PLSR | 0.51 | — | Feature selection via VIP |
| Random Forest | 0.66 | — | Nonlinear capture |
| ANN | 0.05 | — | ❌ Insufficient data |
| GPR | **0.75** | — | Uncertainty quantification |
| Hybrid | 0.72 | — | Domain knowledge integration |
| XGBoost+SHAP | 0.74 | — | Explainability + accuracy |

### BMS2 (N=10,000)

| Model | R² | Δ from BMS1 | Key Strength |
|-------|:--:|:----------:|-------------|
| Ridge | 0.57 | +11.8% | Interpretability |
| PLSR | 0.57 | +11.8% | Feature selection via VIP |
| Random Forest | 0.79 | +19.7% | Robust across data regimes |
| ANN | 0.77 | +1440% | Scalable with data |
| GPR | 0.79 | +5.3% | Strong RBF kernel prior |
| Hybrid | 0.83 | +15.3% | Physics-informed features |
| XGBoost+SHAP | **0.83** | +12.2% | Best overall — production ready |

---

## 🤖 Automated Data Cleansing Module

An end-to-end automation pipeline that takes **any raw dataset** and prepares it for the 7-model comparison:

**What it does:**
- **Missing value handling** — detection, imputation (mean/median/KNN), and reporting
- **Outlier detection & treatment** — IQR and z-score based filtering
- **Feature type inference** — automatic numeric/categorical detection and encoding
- **Scaling & normalization** — StandardScaler, MinMaxScaler based on model requirements
- **Feature engineering** — auto-generates interaction terms and polynomial features
- **Train/test splitting** — stratified splitting with configurable ratios
- **Pipeline execution** — preprocessed data is automatically fed into all 7 models with results exported for comparison

**Usage:**
```bash
python automation/pipeline_runner.py --input your_dataset.csv --target target_column
```

---

## 📈 Interactive Results Dashboard

A visual dashboard for exploring and comparing model results across both experiments:

**Features:**
- Side-by-side R² comparison across all 7 models (BMS1 vs BMS2)
- Radar charts for multi-criteria model evaluation (accuracy, robustness, interpretability, speed)
- SHAP summary and feature importance visualizations
- Learning curves showing model performance vs training data size
- ANN convergence comparison (N=500 vs N=10,000)
- Noise robustness analysis across models
- Exportable rubric heatmaps for model selection scoring

---

## 🧪 Dataset Design

The synthetic dataset is **mechanistically grounded** in published biochemistry:

**Input Variables:**
| Variable | Role | Reference |
|----------|------|-----------|
| GDP-Fucose | Substrate concentration | Raju et al. 2000 |
| Temperature | Culture condition | Hossler et al. 2009 |
| pH | Culture condition (quadratic effect) | Jedrzejewski et al. 2014 |
| Dissolved Oxygen | Culture condition | — |
| Manganese (Mn) | Cofactor concentration | — |
| + 5 additional bioprocess variables | Various | — |

**Dataset Properties:**
- Batch effects and measurement noise included
- Time-series component for temporal dynamics
- Missing data variants for robustness testing
- Identical ground truth equations across BMS1 and BMS2

**Only 3 parameters changed between experiments:**
| Parameter | BMS1 | BMS2 |
|-----------|:----:|:----:|
| N_SAMPLES | 500 | 10,000 |
| N_BATCHES | 10 | 50 |
| N_BATCHES_TS | 20 | 100 |

This isolates **dataset size as the single experimental variable**.

---

## 💡 Key Takeaways

1. **Linear models hit a structural ceiling** — Ridge and PLSR cannot capture nonlinear relationships regardless of data volume.
2. **Tree-based models are robust across data regimes** — XGBoost and RF perform well even at N=500 and scale with more data.
3. **GPR benefits from strong priors, not data volume** — RBF kernel encodes the right structural assumption, making it the best small-data model.
4. **ANNs require large datasets — period** — with 12,225 parameters and no structural prior, the ANN needs thousands of samples to learn what other models get from architecture.
5. **XGBoost + SHAP is the recommended production model** — consistent top accuracy with full explainability across both experiments.
6. **Physics-informed features reduce dimensionality by 50%** while retaining ~75% of predictive performance — valuable for interpretability and regulatory compliance.

---

## 🏥 Industrial Relevance

- Pharmaceutical bioprocessing datasets are typically **N < 1,000** — making the BMS1 results directly applicable
- Regulatory frameworks (ICH Q8/Q9) require model interpretability — favoring XGBoost+SHAP and GPR over black-box ANNs
- The automated cleansing pipeline enables rapid deployment on new bioprocess datasets without manual preprocessing

---

## 🛠️ Tech Stack

`Python` · `XGBoost` · `SHAP` · `scikit-learn` · `TensorFlow/Keras` · `GPy` · `Pandas` · `NumPy` · `Matplotlib` · `Seaborn` · `LIME`

---

## 🚀 Getting Started

```bash
# Clone the repository
git clone https://github.com/SanjithGanesh/BMS-Fucosylation-Prediction.git
cd BMS-Fucosylation-Prediction

# Install dependencies
pip install -r requirements.txt

# Run BMS1 experiment
cd BMS
python run_all_models.py

# Run BMS2 experiment
cd ../BMS2
python run_all_models.py

# Run automated cleansing + comparison on custom data
python automation/pipeline_runner.py --input your_data.csv --target fucosylation

# Launch dashboard
python dashboard/app.py
```

---

## 📄 Citation

If referencing this work:
```
Ganesh, S. (2025). Predictive Modelling of mAb Fucosylation in CHO Cell Bioprocessing.
MSc Data Science, Rutgers University.
```

---

## 📬 Contact

**Sanjith Ganesh** — [LinkedIn](https://linkedin.com/in/sanjithganesh) · [GitHub](https://github.com/SanjithGanesh) · [Portfolio]([https://sanjithganesh.com](https://sanjithganesh.github.io/Portfolio-Website/)) · gsj2442@gmail.com
