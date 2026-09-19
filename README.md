# 🤖 Google Search Ranking & Discoverability — ML Capstone
### FlyRank AI Machine Learning Internship | July – September 2026

**Content decline scoring model built on 79M+ production rows of real search data.**
Designed to flag web pages showing signs of performance decline and prioritize them for content refresh — beating a hand-written baseline by a wide margin.

🔗 **Full capstone research paper:** [emanhrustemovic.github.io/ml-capstone-paper](https://emanhrustemovic.github.io/ml-capstone-paper)

---

## 📊 Key Results

| Metric | Baseline (Hand-written rule) | ML Model (Logistic Regression) |
|--------|------------------------------|-------------------------------|
| **Precision@50** | 0.70 | **0.96** |
| **AUC** | 0.544 | **0.649** |
| **Pages scored** | — | **92,148** |
| **Dataset size** | — | **79M+ rows** |

> ✅ Results re-validated under a **stricter client-grouped split** to rule out data leakage — the lift holds.

---

## 🎯 Problem Statement

Content teams cannot manually review every page every month. The goal was to build a model that:
- Flags pages with **declining search performance** worth prioritizing for content refresh
- Produces **ranked action recommendations** with reason codes for each flagged page
- Provides **honest, reproducible validation** — not cherry-picked numbers

---

## 🏗️ ML Pipeline

```
Raw Search Data (79M+ rows, DuckDB)
        ↓
Data Contract & Leakage Checks
        ↓
Feature Engineering (CTR, position, trend signals)
        ↓
Hand-written Baseline (CTR-vs-position rule)
        ↓
Logistic Regression Model
        ↓
Client-Grouped Re-validation (honest split)
        ↓
Reason Codes + Ranked Action Playbook (92,148 pages)
        ↓
Capstone Research Paper (publicly deployed)
```

---

## 🔍 Methodology

**Data & Features:**
- Dataset: FlyRank production search warehouse (~79M rows) via DuckDB
- Features: CTR signals, position trends, content freshness indicators
- Anonymized — no client names, domains, or private data

**Validation approach:**
- Standard train/test split → then re-validated with **client-grouped split** to detect leakage
- Precision@K evaluation — ranking the right pages first is what matters
- Reproducible pipeline with fixed random seeds

**Why Logistic Regression?**
- Interpretable — each feature's contribution is explainable
- Fast to train and deploy
- Competitive with more complex models on this task

---

## 📁 Repository Structure

```
├── notebooks/
│   ├── 01_first_look_and_discovery.ipynb    # EDA and data exploration
│   ├── 02_your_first_readable_model.ipynb   # Baseline model
│   └── 03_working_with_the_full_release.ipynb # Full 79M row pipeline
├── scripts/
│   ├── 01_prepare_features.py               # Feature engineering
│   ├── 02_baseline_score.py                 # Hand-written rule baseline
│   ├── 03_train_model.py                    # Model training
│   ├── 04_evaluate_and_export.py            # Evaluation + ranked queue
│   ├── 05_build_pdf_report.py               # PDF report generation
│   └── run_all.py                           # Full pipeline runner
├── data/
│   └── raw/content_refresh_anonymized.csv   # Anonymized starter dataset
├── outputs/
│   ├── model_report.md                      # Model evaluation report
│   ├── refresh_queue_sample.csv             # Ranked action recommendations
│   └── charts/                              # Evaluation visualizations
├── work/                                    # Capstone work directory
└── docs/                                    # Documentation & data dictionary
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Python** | Core language |
| **DuckDB** | Querying 79M+ row production dataset |
| **Pandas / NumPy** | Data manipulation |
| **Scikit-learn** | Model training & evaluation |
| **Hugging Face Datasets** | Dataset access |
| **Matplotlib** | Visualization |

---

## 🚀 How to Run

### Option 1: Google Colab (recommended, zero setup)
Click any notebook badge above and run all cells.

### Option 2: Local

```bash
git clone https://github.com/EmanHrustemovic/FlyRank-AI-Intership-ML-Track-
cd FlyRank-AI-Intership-ML-Track-
pip install -r requirements.txt
python scripts/run_all.py
```

---

## 🏆 Internship Completion

| Detail | Value |
|--------|-------|
| **Program** | FlyRank AI Machine Learning Internship |
| **Track** | Machine Learning Engineering |
| **Period** | July 1 – September 7, 2026 |
| **Assignments completed** | 34 (136.5h) |
| **Capstone** | ✅ Accepted by lead track mentor |
| **Certificate** | [Verify](https://internship.flyrank.ai/verify?id=FR-D10-E9EFE-27EA2) |
| **Mentor** | Mirza Ašćerić — Director of AI Development @ 10x.ai & FlyRank AI |

---

## 👤 Author

**Eman Hrustemović** — Junior Data Scientist & ML Engineer

- 🔗 [GitHub](https://github.com/EmanHrustemovic)
- 🔗 [LinkedIn](https://linkedin.com/in/eman-hrustemovic)
- 📄 [Capstone Paper](https://emanhrustemovic.github.io/ml-capstone-paper)
- 📧 emanhrustemovic6@gmail.com
