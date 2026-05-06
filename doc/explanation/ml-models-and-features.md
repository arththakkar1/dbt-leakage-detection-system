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
*   **Target:** Detecting Middlemen and Undrawn Funds.
*   **Why:** Traditional rule-engines fail when middlemen slightly alter their behavior. An Isolation Forest is an unsupervised machine learning algorithm that identifies anomalies instead of normal observations.
*   **Features Used:** `beneficiaries_per_account`, `days_since_withdrawal`, `amount`.
*   **Logic:** Accounts acting as aggregation nodes for multiple unrelated beneficiaries will be isolated as anomalies in the high-dimensional feature space.

### Model B: Fuzzy String Matching (Transliteration Handling)
*   **Target:** Duplicate Identity Detection.
*   **Why:** Exact string matching fails on Gujarati-to-English transliterations (e.g., *Suresh Patel* vs *Sureshbhai Ptl*).
*   **Algorithm:** `FuzzyWuzzy` library combined with phonetic hashing (Double Metaphone). If two names map to the same phonetic hash, the Levenshtein distance is calculated. 
*   **Threshold:** A similarity score `> 85%` triggers a deeper demographic check (e.g., matching DOB or District).

### Model C: High-Speed Deterministic Heuristics
*   **Target:** Deceased Beneficiaries & Cross-Scheme Claims.
*   **Why:** These do not require probabilistic ML. They require absolute, lightning-fast set intersections.
*   **Algorithm:** Pandas dataframe joins or Redis `SINTER` (Set Intersection) operations.

### Model D: Prescriptive AI (Actionable Suggestions)
*   **Target:** Translating raw classifications or regression scores into actionable, human-readable advice for the District Finance Officer.
*   **Why:** Simply returning a classification flag (e.g., `MIDDLEMAN_ACCOUNT`) leaves the officer wondering what to do next. We use a model to explicitly state the fault and suggest a remedy.
*   **Algorithm:** An LLM (Large Language Model) or a Rule-Based Expert System. It takes the outputs of Models A, B, and C as its context prompt and generates a prescriptive sentence.

## 4. Predicted Values (Outputs)

The ML Engine does not operate as a "black box". It returns highly structured, explainable predictions.

### 1. Classification Flags
The system predicts the specific category of fraud detected:
*   `DECEASED_DISBURSEMENT`
*   `DUPLICATE_IDENTITY_TRANSLITERATION`
*   `CROSS_SCHEME_VIOLATION`
*   `MIDDLEMAN_ACCOUNT`
*   `DORMANT_UNDRAWN_FUNDS`

### 2. Risk Score (0-100)
A deterministic probability score indicating the severity and confidence of the anomaly. 
*   **0-30:** Low Risk (Cleared)
*   **31-70:** Medium Risk (Flagged for routine review)
*   **71-100:** Critical Risk (Funds frozen, immediate field verification required)

### 3. Evidence String (Explainable AI)
Every prediction includes a human-readable explanation string, for example: 
> *"Aadhaar 1234 matched against Death Register. Transaction date (2024-02-12) occurred 45 days after Official Death Date (2023-12-29)."*

### 4. Prescriptive Suggestion (Model D Output)
Beyond simple classification, the system provides a direct suggestion on how to handle the fault.
> *"Suggestion: Immediately halt funds. Do not dispatch field verifier as death is confirmed by State Registry."*

## 5. End-to-End Example Flow

To illustrate how features become predictions:

**Scenario: Suspected Middleman Interception**
*   **Raw Inputs:** Transaction `txn_882` for "Ramesh Patel", Amount: ₹2000. Target Bank Account: `000123456`.
*   **Feature Extraction (Inputs):** The engine queries the database and calculates `beneficiaries_per_account = 6`. It calculates `days_since_withdrawal = 180`.
*   **Model Processing:** The Isolation Forest (Model A) evaluates `[6, 180, 2000]` and isolates this point as a severe anomaly compared to the normal cluster (Score: 0.89).
*   **Predicted Classification:** `MIDDLEMAN_ACCOUNT`
*   **Predicted Risk Score:** `94` (Critical)
*   **Generated Suggestion (Model D):** *"Fault: Account 000123456 is receiving funds for 6 distinct beneficiaries and no withdrawals have occurred in 180 days. Suggestion: Freeze account immediately and dispatch Verifier to Ramesh's registered address to investigate potential passbook confiscation by an agent."*
