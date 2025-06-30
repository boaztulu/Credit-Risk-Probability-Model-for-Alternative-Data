<!-- ============================================================= -->
<!--  README  ✨ Credit-Risk Probability Model for Alternative Data -->
<!-- ============================================================= -->

<h1 align="center">💳&nbsp; Credit-Risk Probability Model <br/>for <i>Alternative Data</i> 🚀</h1>

<p align="center">
  <b>Bati Bank × e-Commerce BNPL Pilot</b><br/>
  <em>From raw transactions → MLflow-versioned model → FastAPI micro-service</em>
</p>

---

## 🏷️ <big><b>1&nbsp;·&nbsp;Business Context</b></big>  <sup>(“Credit Scoring Business Understanding”)</sup>

| ✅ Requirement | 🏛️ Basel II Take-away | 🛠️ What We Implement |
|---------------|-----------------------|----------------------|
| **Interpretability & Audit** | Risk estimates must be explainable and traceable. | Modular pipeline → EDA ▶︎ feature scripts ▶︎ MLflow runs ▶︎ FastAPI. |
| **No labelled defaults** | e-Commerce data lacks true default tags. | Engineer RFM-based **proxy** label `is_high_risk`. |
| **Model risk vs. performance** | Simple scorecards = easy validation; GBMs = AUC boost. | Train **LogReg** ⚖️ **GBM**, compare in MLflow, register champion. |

---

## 📊 <big><b>2&nbsp;·&nbsp;Exploratory Data Analysis</b></big>  *(see `notebooks/1.0-eda.ipynb`)*

| 🔍 Insight | 🔗 Evidence |
|-----------|------------|
| **Spend is ultra right-skewed** ( 80 % \< UGX 250 k ) | Log-histogram + summary stats |
| **Three channels dominate** ( web, iOS, Android → 94 % ) | Interactive pie chart |
| **Fraud is < 0.3 %** and voided same day | Grouped bar chart |
| **Recency ⊖ Monetary** ρ ≈ –0.58 | Correlation heat-map |

---

## 🛠️ <big><b>3&nbsp;·&nbsp;Feature Engineering</b></big>

* `RFMAggregator` ➜ **one-row-per-CustomerId**  
* Pipeline ➡️ `median-impute → StandardScaler → One-Hot`  
* Unit tests in `tests/` ensure deterministic transforms.

---

## 🔐 <big><b>4&nbsp;·&nbsp;Proxy Target Engineering</b></big>

1. Compute **R · F · M** features.  
2. `KMeans(k=3)` on scaled RFM.  
3. Cluster with lowest F & M ⇒ `is_high_risk = 1`.  
4. Class balance ≈ **19.7 %** high-risk 🟥  

---

## 🎯 <big><b>5&nbsp;·&nbsp;Model Training & Tracking</b></big>

| Step | Details |
|------|---------|
| **Algorithms** | LogisticRegression (**liblinear, balanced**) & GradientBoostingClassifier |
| **Tuning** | `GridSearchCV` (scoring = ROC-AUC, cv = 5) |
| **Metrics** | ROC-AUC · Accuracy · Precision · Recall · F1 |
| **MLflow** | Each trial logs 🔑 params + 📈 metrics + artifacts. Best model tagged **`champion`**. |

---

## 🌐 <big><b>6&nbsp;·&nbsp;Model Serving</b></big>

```mermaid
graph LR
  Browser[Client] -- REST --> FastAPI
  FastAPI -- loads on startup --> MLflow[(Model Registry)]
  FastAPI -- /predict --> ChampionModel
credit-risk-model/
├── data/            📁 raw & processed
├── notebooks/       🧪 Jupyter EDA
├── src/
│   ├── data_processing.py   ⚙️  features
│   ├── target_engineering.py⚙️  proxy label
│   ├── train.py             🤖 training
│   └── api/                 🌐 FastAPI app
├── tests/           ✅ unit tests
├── Dockerfile       🐳
├── docker-compose.yml
└── README.md        📜 you are here


# 1⃣  Install deps
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 2⃣  Feature engineering
python -m src.data_processing \
       --raw data/raw/transactions.csv \
       --out data/processed/features.parquet

# 3⃣  Build proxy labels
python -m src.target_engineering \
       --raw data/raw/transactions.csv \
       --out data/processed/high_risk_labels.parquet

# 4⃣  Merge & train
python -m src.train --features data/processed/features_with_target.parquet

# 5⃣  Serve champion model
docker compose up --build
# → open http://localhost:8000/docs 🚀




> **Tip:** GitHub markdown renders `<big>` tags and emojis inline, providing the desired “font size” variation without breaking compatibility.
