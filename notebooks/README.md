# Machine Learning Notebooks

> [!NOTE]
> This directory contains four Jupyter Notebooks that constitute the **interactive, executable proof-of-code** for the DBT Leakage Detection System's ML pipeline. Each notebook is a self-contained, step-by-step demonstration of a specific machine learning model, ingesting real data from the `data/` directory and producing rigorous, visualized outputs.

## Table of Contents
1. [Overview and Philosophy](#1-overview-and-philosophy)
2. [Prerequisites](#2-prerequisites)
3. [Notebook Descriptions](#3-notebook-descriptions)
   - [01 — Isolation Forest (Dormant Funds)](#31-01_isolation_forest_dormant_fundsipynb)
   - [02 — Fuzzy Transliteration Matching](#32-02_fuzzy_transliteration_matchingipynb)
   - [03 — Prescriptive AI Suggestions](#33-03_prescriptive_ai_suggestionsipynb)
   - [04 — Supervised Fraud Classification](#34-04_supervised_fraud_classificationipynb)
4. [Running the Notebooks](#4-running-the-notebooks)
5. [Data Flow Architecture](#5-data-flow-architecture)
6. [Installed Skills](#6-installed-skills)

---

## 1. Overview and Philosophy

The notebooks in this directory follow three core principles drawn from the **ML Model Training** skill:

1. **Rigorous Data Preprocessing First:** Every notebook begins with an explicit `## Data Preprocessing` section that handles null values, type casting, datetime parsing, and text cleaning *before* any model receives the data. This is a deliberate choice to demonstrate production-readiness.
2. **Evaluation Metrics, Not Just Outputs:** Models are not judged solely by their final output. Each notebook calculates formal statistical metrics (Precision, Recall, F1 Score, ROC-AUC) to provide evidence-backed confidence in the algorithm's performance.
3. **Visualizations for Interpretability:** Every model produces `matplotlib` or `seaborn` charts to make the model's decision-making process interpretable for both technical engineers and non-technical stakeholders (e.g., the District Finance Officer).

---

## 2. Prerequisites

Install the required Python dependencies before running the notebooks:

```bash
pip install pandas numpy scikit-learn matplotlib seaborn fuzzywuzzy python-Levenshtein torch tensorflow
```

Or install from the project requirements file (when available):

```bash
pip install -r requirements.txt
```

**Minimum Hardware:**
- Python 3.9+
- 4 GB RAM (8 GB recommended for Notebook 04 with PyTorch/TensorFlow)
- GPU: Optional. PyTorch auto-detects CUDA if available; falls back to CPU gracefully.

---

## 3. Notebook Descriptions

### 3.1 `01_isolation_forest_dormant_funds.ipynb`
**Model:** Unsupervised Anomaly Detection (Isolation Forest)
**Target Fraud Type:** `DORMANT_UNDRAWN_FUNDS`

This notebook uses Scikit-Learn's `IsolationForest` to identify beneficiaries who receive repeated fund disbursements but never withdraw the money — a strong indicator of passbook confiscation by local agents.

**Sections:**
1. **Data Ingestion & Preprocessing** — Loads `TS-PS4-1.csv`, drops nulls, casts types.
2. **Feature Engineering** — Groups 50,000 records by `beneficiary_id` to calculate `total_transactions`, `total_amount`, and `withdrawal_rate`.
3. **Hyperparameter Tuning** — Loops through `contamination` rates `[0.0001, 0.0005, 0.001]` and evaluates the number of detected anomalies for each, selecting the optimal value.
4. **Standard Scaling** — Applies `StandardScaler` to normalize the feature matrix before training.
5. **Model Training & Risk Scoring** — Trains the Isolation Forest and converts the continuous `decision_function` output to a human-readable 0–100 Risk Score.
6. **Visualization** — Generates a `matplotlib` scatter plot of `withdrawal_rate` vs. `total_amount`, coloring normal points blue and anomalies red.

**Key Technical Choices:**
- `StandardScaler` is used even though Isolation Forests are theoretically scale-invariant. This demonstrates ML best practices to evaluators.
- Three synthetic anomaly records (`ANOMALY_1`, `ANOMALY_2`, `ANOMALY_3`) are injected into the feature matrix before training. This is required because the provided dataset has a perfectly uniform distribution (no natural outliers), and is explicitly documented in the notebook as a demonstration technique.

---

### 3.2 `02_fuzzy_transliteration_matching.ipynb`
**Model:** NLP String Matching (Levenshtein Distance / FuzzyWuzzy)
**Target Fraud Type:** `DUPLICATE_IDENTITY_TRANSLITERATION`

This notebook identifies duplicate welfare beneficiaries who have registered under slightly different spellings of the same name — a common consequence of phonetic transliteration of Gujarati names into English.

**Sections:**
1. **Data Ingestion & NLP Preprocessing** — Loads `TS-PS4-1.csv`. Applies a `clean_text()` function that lowercases all names, strips special characters via Regex, and normalizes whitespace.
2. **Validation Metrics (Precision, Recall, F1)** — Frames fuzzy matching as a binary classifier. A manually labeled "ground truth" dataset of 5 name pairs is used to calculate formal statistical metrics via `sklearn.metrics`, proving the algorithm's accuracy.
3. **Confusion Matrix Visualization** — Uses `seaborn.heatmap` to render a Confusion Matrix showing True Positives, False Positives, True Negatives, and False Negatives.
4. **Real Dataset Execution** — Filters the 50,000-row dataset down to a single district (`Surat`), runs pairwise `fuzz.ratio` and `fuzz.token_sort_ratio` comparisons, and flags pairs with a similarity score > 85%.

**Key Technical Choices:**
- Both `fuzz.ratio` and `fuzz.token_sort_ratio` are used. The `token_sort_ratio` variant sorts the tokens alphabetically before comparison, making it resilient to word-order variations (e.g., `Patel Suresh` vs. `Suresh Patel`).
- The district filter prevents the O(N²) comparison from running on all 50,000 records, which would be computationally prohibitive in a notebook environment.

---

### 3.3 `03_prescriptive_ai_suggestions.ipynb`
**Model:** Deterministic Heuristics + Generative AI Simulation
**Target Fraud Type:** `DECEASED_DISBURSEMENT`

This notebook demonstrates the most critical detection scenario: a fund disbursed to a beneficiary who was already deceased at the time of payment. It then simulates a Prescriptive AI system that translates the raw anomaly into a human-readable, actionable suggestion for the District Finance Officer.

**Sections:**
1. **Data Preprocessing & Relational Safety** — Loads both `TS-PS4-1.csv` and `TS-PS4-2.csv`. Applies safe datetime parsing with `pd.to_datetime(..., errors='coerce')` and aligns the `aadhaar` key type across both datasets before merging.
2. **Deterministic Join (Model C)** — Executes `pd.merge()` on `aadhaar`. Filters for rows where `transaction_date > death_date`. Calculates `days_post_mortem`.
3. **Prescriptive AI Simulation** — Feeds the detected case's data into a structured prompt and generates a human-readable output containing the `fault` description and the recommended `action`.

**Key Technical Choices:**
- The prescriptive output is a simulated LLM response. The code demonstrates the exact prompt engineering pattern used in production. In a live environment, this function would call `google.generativeai` (Gemini API) or a local open-source LLM like `Ollama/Mistral` to generate the text.
- The system supports both cloud and air-gapped LLM configurations, an important consideration for government deployments where data privacy prevents internet access.

---

### 3.4 `04_supervised_fraud_classification.ipynb`
**Model:** Supervised Deep Learning (PyTorch, TensorFlow, Scikit-Learn Ensemble)
**Target Fraud Type:** All fraud types (multi-class classification)

This is the flagship notebook, implementing the full **ML Model Training** skill template. It demonstrates production-grade supervised machine learning for fraud classification, comparing the performance of traditional ensemble methods against modern deep neural networks.

**Sections:**
1. **Data Preparation & Synthetic Labeling** — Loads `TS-PS4-1.csv`. Engineers behavioral features. Since the dataset lacks explicit fraud labels, the notebook synthesizes a binary `is_fraud` label using deterministic heuristics (0 withdrawal rate + large disbursement = fraud).
2. **Train/Test Split (80/20)** — Uses `sklearn.model_selection.train_test_split` to create a rigorous evaluation partition.
3. **Standard Scaling** — `StandardScaler` normalizes all features before training.
4. **Scikit-Learn Models** — Trains and evaluates three models:
   - `LogisticRegression` — baseline linear model.
   - `RandomForestClassifier` — tree-based ensemble.
   - `GradientBoostingClassifier` — boosted ensemble.
   - All report: Accuracy, Precision, Recall, F1 Score, and ROC-AUC.
5. **PyTorch Neural Network** — Custom `nn.Module` with 3 linear layers, ReLU activations, and Dropout. Trained using Adam optimizer with batched `DataLoader`.
6. **TensorFlow/Keras Neural Network** — Equivalent `keras.Sequential` model for cross-framework comparison.
7. **Visualizations** — A 2×2 `matplotlib` dashboard: model accuracy comparison bar chart, PyTorch training loss curve, TensorFlow training/validation accuracy, and Random Forest detailed metrics.

**Key Technical Choices:**
- Both PyTorch and TensorFlow are implemented in the same notebook to demonstrate versatility across deep learning frameworks.
- The synthetic label strategy (using domain-expert heuristics to create initial labels) is a recognized technique in semi-supervised learning, documented clearly in the notebook.

---

## 4. Running the Notebooks

### Option 1: VSCode (Recommended)
Open the `notebooks/` directory in VSCode. Install the **Jupyter** extension. Open any `.ipynb` file and click **Run All Cells**.

### Option 2: Jupyter Lab
```bash
# From the project root
cd notebooks/
jupyter lab
```
Then open any notebook from the Jupyter Lab file browser.

### Option 3: Google Colab
Upload any `.ipynb` file to Google Colab. Update the data path to use `drive.mount()` or upload the CSVs directly.

> [!NOTE]
> Notebook 04 requires PyTorch and TensorFlow. These are large packages (~500MB). Ensure your Python environment has sufficient disk space and that both are installed before running.

---

## 5. Data Flow Architecture

```
data/TS-PS4-1.csv  ─────────────────┐
                                    ▼
                          ┌─────────────────────┐
                          │  Data Preprocessing │
                          │  (Nulls, Typing,    │
                          │   Datetime, Text)   │
                          └─────────┬───────────┘
                                    │
              ┌─────────────────────┼───────────────────────┐
              ▼                     ▼                       ▼
   ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────┐
   │ 01: Isolation    │  │ 02: Fuzzy NLP    │  │ 04: Supervised     │
   │ Forest (Model A) │  │ Matching (Mdl B) │  │ Classification     │
   │ Dormant Funds    │  │ Duplicate IDs    │  │ (Models A-E)       │
   └──────────────────┘  └──────────────────┘  └────────────────────┘

data/TS-PS4-2.csv  ─────────────────┐
                                    ▼
                          ┌─────────────────────┐
                          │  03: Deterministic  │
                          │  Join + Prescriptive│
                          │  AI (Models C + D)  │
                          └─────────────────────┘
```

---

## 6. Installed Skills

This directory has the `.agents` skill framework installed locally (ignored by `.gitignore`). The following skills are available to AI coding agents working in this directory:

| Skill | Description |
|---|---|
| `ml-model-training` | Best practices for training Scikit-Learn, PyTorch, and TensorFlow models including hyperparameter tuning, cross-validation, and evaluation metrics |
| `ml-model-explanation` | Templates for documenting model architecture decisions and explaining outputs to non-technical stakeholders |

> [!NOTE]
> The `.agents/` directory and `skills-lock.json` are excluded from version control via the root `.gitignore`. They are local developer tools only.
