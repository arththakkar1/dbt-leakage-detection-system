# Machine Learning Models and Data Features

> [!NOTE]
> This document provides a deep dive into the specific machine learning models, 
> heuristics, data features, and predicted outputs used by the Python ML Engine.

## Table of Contents
- [1. Data Sources and Context](#1-data-sources-and-context)
- [2. Feature Engineering (Inputs)](#2-feature-engineering-inputs)
- [3. The ML Models and Rule Engines](#3-the-ml-models-and-rule-engines)
- [4. Predicted Values (Outputs)](#4-predicted-values-outputs)

## 1. Data Sources and Context

The ML engine ingests simulated transaction and vital statistics data to identify fraudulent disbursement patterns.

*   **Primary Transactions (`TS-PS4-1.csv`)**: Contains individual disbursement records. Key fields include `aadhaar`, `name`, `scheme`, `amount`, `transaction_date`, and `withdrawn`.
*   **Civil Death Register (`TS-PS4-2.csv`)**: The ground-truth vital statistics registry. Key fields include `aadhaar` and `death_date`.

## 2. Feature Engineering (Inputs)

Before the models can evaluate a transaction, the Python engine extracts and engineers specific features from the raw JSON payload:

### Temporal Features
*   `days_since_withdrawal`: Calculated from the `withdrawn` status over historical transactions. High values indicate dormant funds.
*   `days_since_death`: Calculated as `transaction_date` - `death_date`. If negative, the transaction occurred after the beneficiary's death.

### Graph & Relational Features
*   `beneficiaries_per_account`: A count of distinct `beneficiary_id` hashes tied to a single `bank_account_no`. Used to detect middlemen.
*   `active_schemes_count`: The number of mutually exclusive schemes linked to a single `aadhaar`.

### NLP & Phonetic Features
*   `phonetic_hash`: The Double Metaphone representation of the Gujarati name.
*   `levenshtein_distance`: The calculated string distance between two names within the same payment batch.

## 3. The ML Models and Rule Engines

The system employs an ensemble of algorithms to ensure both high accuracy and computational speed.

### Model A: Isolation Forests (Behavioral Anomaly Detection)
*   **Target:** Detecting Dormant Funds and Undrawn Anomalies.
*   **Why:** Traditional rule-engines fail when behaviors change subtly. An Isolation Forest is an unsupervised machine learning algorithm that identifies mathematical anomalies instead of normal observations.
*   **Features Used:** `total_transactions`, `withdrawal_rate`, `total_amount`.
*   **Logic:** Beneficiaries receiving massive disbursements but maintaining a `0.0` withdrawal rate are isolated as extreme anomalies in the high-dimensional feature space, triggering flags.

### Model B: Fuzzy String Matching (Transliteration Handling)
*   **Target:** Duplicate Identity Detection.
*   **Why:** Exact string matching fails on Gujarati-to-English transliterations (e.g., *Suresh Patel* vs *Sureshbhai Ptl*).
*   **Algorithm:** `FuzzyWuzzy` library combined with phonetic hashing (Double Metaphone). If two names map to the same phonetic hash, the Levenshtein distance is calculated. 
*   **Threshold:** A similarity score `> 85%` triggers a deeper demographic check (e.g., matching DOB or District).

### High-Speed Deterministic Heuristics
*   **Target:** Deceased Beneficiaries & Cross-Scheme Claims.
*   **Why:** These do not require probabilistic ML. They require absolute, lightning-fast set intersections.
*   **Algorithm:** Pandas dataframe joins or Redis `SINTER` (Set Intersection) operations.

### Model C: Prescriptive AI (Actionable Suggestions)
*   **Target:** Translating raw classifications or regression scores into actionable, human-readable advice for the District Finance Officer.
*   **Why:** Simply returning a classification flag (e.g., `DORMANT_UNDRAWN_FUNDS`) leaves the officer wondering what to do next. We use a generative model to explicitly state the fault and suggest a remedy.
*   **Algorithm:** An LLM (Large Language Model) or a Rule-Based Expert System. It takes the outputs of the deterministic engines as its context prompt and generates a prescriptive sentence.

### Model D: Supervised Fraud Classification (Deep Learning)
*   **Target:** Production-grade predictive classification of fraudulent patterns.
*   **Why:** To catch complex, non-linear relationships between variables that simple heuristics miss.
*   **Algorithm:** We utilize **PyTorch** and **TensorFlow/Keras** Neural Networks alongside Scikit-Learn ensembles (Random Forest, Gradient Boosting). Features are standardized using `StandardScaler` and evaluated using K-Fold cross-validation, plotting Accuracy, Precision, Recall, F1, and ROC-AUC via `matplotlib`.

## 4. Predicted Values (Outputs)

The ML Engine does not operate as a "black box". It returns highly structured, explainable predictions.

### 1. Classification Flags
The system predicts the specific category of fraud detected:
*   `DECEASED_DISBURSEMENT`
*   `DUPLICATE_IDENTITY_TRANSLITERATION`
*   `CROSS_SCHEME_VIOLATION`
*   `DORMANT_UNDRAWN_FUNDS`

### 2. Risk Score (0-100)
A deterministic probability score indicating the severity and confidence of the anomaly. 
*   **0-30:** Low Risk (Cleared)
*   **31-70:** Medium Risk (Flagged for routine review)
*   **71-100:** Critical Risk (Funds frozen, immediate field verification required)

### 3. Evidence String (Explainable AI)
Every prediction includes a human-readable explanation string, for example: 
> *"Aadhaar 1234 matched against Death Register. Transaction date (2024-02-12) occurred 45 days after Official Death Date (2023-12-29)."*

### 4. Prescriptive Suggestion (Model C Output)
Beyond simple classification, the system provides a direct suggestion on how to handle the fault.
> *"Suggestion: Immediately halt funds. Do not dispatch field verifier as death is confirmed by State Registry."*

## 5. End-to-End Example Flow

To illustrate how features become predictions:

**Scenario: Suspected Dormant Fund / Passbook Confiscation**
*   **Raw Inputs:** Transaction `txn_882` for "Ramesh Patel", Amount: ₹50000 across 10 transactions.
*   **Feature Extraction (Inputs):** The engine calculates `total_transactions = 10`, `total_amount = 50000`, and `withdrawal_rate = 0.0`.
*   **Model Processing:** The Isolation Forest (Model A) evaluates `[10, 50000, 0.0]` and isolates this point as a severe anomaly compared to the normal cluster (Score: 0.91).
*   **Predicted Classification:** `DORMANT_UNDRAWN_FUNDS`
*   **Predicted Risk Score:** `91` (Critical)
*   **Generated Suggestion (Model C):** *"Fault: Beneficiary has received ₹50,000 but the withdrawal rate is 0%. Suggestion: Freeze account immediately and dispatch Verifier to Ramesh's registered address to investigate potential passbook confiscation by an agent."*
