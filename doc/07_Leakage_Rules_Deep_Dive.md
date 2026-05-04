# DBT Leakage Detection System: Leakage Rules & Risk Scoring Math

This document details the exact logical and mathematical frameworks used by the Python ML service to detect anomalies and assign Risk Scores.

## 1. The 4 Leakage Detection Rules

### Rule 1: Deceased Beneficiary Detection
*   **Logic:** Exact or fuzzy match of `aadhaar_hash` and `dob` against the State Civil Death Register.
*   **Exception Handling:** If `aadhaar_hash` is missing, the system uses a composite key of `name_english` + `district_id` + `dob`.
*   **Trigger:** Date of Death (DoD) is strictly before the Transaction Timestamp.

### Rule 2: Duplicate Identity (Transliteration-Aware)
*   **Logic:** Detects when a single person is enrolled twice under slight name variations to bypass exact-match constraints.
*   **Algorithm:**
    1.  Convert Gujarati input strings to phonetic representations using a modified Double Metaphone algorithm.
    2.  Calculate the Levenshtein distance between strings within the same scheme disbursement batch.
*   **Threshold:** A Levenshtein similarity score of `> 85%` combined with identical `address` or `dob` triggers the flag.
*   **Example:** "Patel Ramesh" vs "Patil Rames" -> Flagged.

### Rule 3: Cross-Scheme Duplication
*   **Logic:** Identifies identical `aadhaar_hash` or `bank_account_no` receiving funds from mutually exclusive schemes.
*   **Example Rule:** Scheme A (Unemployment Allowance) and Scheme B (Disability Pension) cannot be claimed by the same individual concurrently.
*   **Trigger:** A simple database intersection operation between active beneficiaries of exclusive schemes.

### Rule 4: Undrawn Funds / Middlemen Detection
*   **Logic (Undrawn):** Queries bank API data (simulated). If `Current_Balance >= (Disbursement_Amount * 3)` and `Last_Withdrawal_Date > 120 days`, it flags the account as dormant, indicating the actual beneficiary is likely unaware of the transfers.
*   **Logic (Middlemen):** Groups `bank_account_no` across all transactions. If `Count(Distinct beneficiary_id) > 2` for a single account (excluding known joint-family accounts), it flags the account as a potential middleman/agent intercepting funds.

## 2. Risk Score Math (Explainable AI)

The system avoids "black box" AI. Every flagged transaction is assigned a Risk Score from 0 to 100 based on a deterministic, additive formula.

**Base Formula:**
`Total Risk Score = Base Violation Score + Evidence Multipliers`

**Example 1: Transliteration Match**
*   Base Violation (Duplicate Name in Scheme): `+40`
*   Levenshtein similarity is 95%: `+30`
*   Bank accounts belong to same branch: `+15`
*   **Total Score: 85 (High Risk)**

**Example 2: Middlemen Detection**
*   Base Violation (Multiple IDs to 1 Bank): `+60`
*   3 Distinct Beneficiaries linked: `+10`
*   5 Distinct Beneficiaries linked: `+30` (Replaces the +10)
*   Aadhaar addresses are in different districts: `+10`
*   **Total Score: 100 (Critical Risk)**

**Example 3: Deceased Detection**
*   Aadhaar Hash Match in Death Register: `+100` (Automatic Critical Flag)
