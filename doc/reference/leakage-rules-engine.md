# Leakage Rules Engine

> [!NOTE]
> Details the logical and mathematical frameworks used by the Python ML service 
> to detect anomalies and assign Risk Scores.

## Table of Contents
- [1. Leakage Detection Rules](#1-leakage-detection-rules)
- [2. Risk Score Math](#2-risk-score-math)

## 1. Leakage Detection Rules

### Rule 1: Deceased Beneficiary Detection
*   **Logic:** Exact match of `aadhaar` against the State Civil Death Register.
*   **Trigger:** Transaction `transaction_date` is strictly after the `death_date`.

### Rule 2: Duplicate Identity (Transliteration-Aware)
*   **Logic:** Detects when a single person is enrolled twice under slight name variations.
*   **Algorithm:** 
    1. Extract phonetic representations (Double Metaphone).
    2. Calculate Levenshtein distance between strings in the same district/scheme.
*   **Threshold:** Levenshtein similarity score `> 85%` triggers a flag.

### Rule 3: Cross-Scheme Duplication
*   **Logic:** Identifies identical `aadhaar` receiving funds from mutually exclusive schemes.
*   **Trigger:** Database intersection between active beneficiaries of exclusive schemes.

### Rule 4: Undrawn / Dormant Funds Detection
*   **Logic:** Checks the `withdrawn` boolean flag across a beneficiary's entire transaction history.
*   **Trigger:** If a beneficiary receives large disbursements (`total_amount > 5000`) but maintains a `withdrawal_rate = 0.0` over multiple transactions, it signals passbook confiscation or dormant funds.

## 2. Risk Score Math

Every flagged transaction is assigned a Risk Score from 0 to 100.

For deterministic rules, this is an additive formula:
`Total Risk Score = Base Violation Score + Evidence Multipliers`

For machine learning models (Isolation Forests), the score is calculated dynamically based on the algorithm's decision function:
`Risk Score = ((max_score - raw_score) / (max_score - min_score)) * 100`

### Example: Transliteration Match
*   Base Violation (Duplicate Name in Scheme): `+40`
*   Levenshtein similarity is 95%: `+30`
*   Same District (`district_id` match): `+15`
*   **Total Score: 85 (High Risk)**

### Example: Deceased Detection
*   Aadhaar Match in Death Register (`transaction_date > death_date`): `+100` 
*   **Total Score: 100 (Critical Risk - Auto Block)**
