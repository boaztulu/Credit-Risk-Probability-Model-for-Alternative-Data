# 📈 Credit-Risk Probability Model for Alternative Data  
*An End-to-End Pipeline for Building, Deploying & Automating a Credit-Scoring System*

---

## 🗂️ Project Overview
Bati Bank is partnering with an e-commerce platform to launch a *Buy-Now-Pay-Later* (BNPL) service.  
Our objective is to transform raw behavioral transaction data into:

1. **A risk-probability score** for each prospective customer.  
2. **A credit scorecard** that maps risk probabilities onto an interpretable scale.  
3. **Optimal loan amount & duration suggestions** that align with the bank’s risk appetite.

The repository follows a strict MLOps-ready structure (`src/`, `tests/`, `Dockerfile`, CI workflow), ensuring reproducibility, automated testing, and seamless deployment.

---

## 📚 Credit Scoring Business Understanding

### 1️⃣ Basel II & the Imperative of Interpretability 🔍  
Basel II allows banks to employ *internal* models for Probability-of-Default (PD) estimation but demands **rigorous validation, documentation, and audit-readiness**.  
Hence, our model must be:  

| Requirement | Implication for the Project |
|-------------|-----------------------------|
| *Transparency* | Prefer algorithms whose decision logic can be articulated to regulators and senior risk managers. |
| *Auditability* | Maintain complete documentation: feature definitions, data lineage, model assumptions, and validation evidence. |
| *Controllability* | Enable easy sensitivity analysis so that risk officers can stress-test model outputs under adverse scenarios. |

A logistic-regression scorecard, for example, satisfies these needs; a purely black-box model would **hinder regulatory approval** and erode institutional trust.

---

### 2️⃣ Proxy Default Variable: Necessity & Risks ⚠️  
Because the dataset lacks an explicit “default” flag, we must engineer a **proxy target** (e.g., ≥90 days past due). This step is essential for supervised learning, yet it introduces three principal risks:

* **Representation Risk** – The proxy may diverge from *true* default behaviour, leading to sub-optimal risk segmentation.  
* **Bias & Misclassification Risk** – A mis-specified proxy can inflate *false positives* (rejecting creditworthy applicants) or *false negatives* (approving risky ones).  
* **Regulatory Scrutiny Risk** – Supervisors may challenge the validity of decisions driven by a proxy; we must justify its economic rationale and statistical soundness.

Mitigation strategies include periodic back-testing against realized defaults and transparent communication of proxy limitations to stakeholders.

---

### 3️⃣ Simple vs. Complex Models: Trade-Offs in a Regulated Context ⚖️  

| 🔑 Dimension | **Simple / Interpretable**<br>*Logistic Regression + Weight-of-Evidence* | **Complex / High-Performance**<br>*Gradient Boosting (GBM)* |
|--------------|------------------------------------------------------------|-------------------------------------------------------------|
| **Interpretability** | ★★★★★ Users, auditors, and customers can trace each coefficient to a risk factor. | ★☆☆☆☆ Requires post-hoc explainers (SHAP, LIME); still opaque to lay readers. |
| **Predictive Power** | ★★★☆☆ Captures linear effects; limited interactions. | ★★★★★ Exploits non-linearities and high-order interactions, improving AUC / Gini. |
| **Regulatory Burden** | Low — decades of acceptance in credit scoring practice. | High — demands extensive documentation, challenger models, and fairness analysis. |
| **Operational Stability** | Robust to data drift; coefficients change gradually. | Sensitive to feature drift; needs tighter monitoring & recalibration. |
| **Maintenance Cost** | Minimal — scorecard updates are straightforward. | Significant — hyper-parameter tuning, model-explainability pipeline, monitoring dashboards. |

**Balanced Strategy 📝**  
*Adopt the simplest model that meets performance thresholds.*  
Deploy GBM only if it yields **material** gains (> ~3–5 pp AUC/Gini) *and* if explainability tooling can satisfy regulatory and ethical standards.

---

> **Key Takeaway 💡** — Basel II governance, the uncertainty of proxy labels, and the heightened cost of model risk management favour *interpretable* solutions for initial production. Complex models remain valuable as challenger models or ensemble components once an explainability framework is in place.

---

## 🏗️ Repository Structure (excerpt)

```text
credit-risk-model/
├── data/
│   ├── raw/              # 📥 original Xente transactions
│   └── processed/        # 🛠️ model-ready features
├── notebooks/
│   └── 1.0-eda.ipynb     # 🔎 exploratory data analysis
├── src/
│   ├── data_processing.py # ⚙️ feature pipelines
│   ├── train.py           # 🎯 model training
│   └── api/               # 🌐 FastAPI service
├── tests/                # ✅ unit tests
└── .github/workflows/ci.yml  # 🤖 automated lint & test
