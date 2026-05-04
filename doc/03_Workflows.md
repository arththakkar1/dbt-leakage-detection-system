# DBT Leakage Detection System: Data Processing & Operational Workflows

This document details the step-by-step workflows for data ingestion, anomaly detection processing, and case resolution.

## 1. Core Data Ingestion and Processing Workflow

This workflow represents the high-speed backend process required to handle 10,000+ transactions in under 30 seconds.

### Phase 1: Data Ingestion
1.  **Trigger:** Scheduled cron job or API trigger signaling a new batch of pre-disbursement DBT data.
2.  **Input:** System receives a payload containing simulated DBT payment records across 3 schemes.
3.  **Validation:** Basic schema validation ensures mandatory fields (Aadhaar, Account Number, Name, Scheme ID) are present.
4.  **Reference Data Loading:** The system securely loads the latest snapshot of the Civil Death Register and Aadhaar linkage mapping into fast in-memory storage (e.g., Redis).

### Phase 2: Parallel Anomaly Detection (Rule Engine)
The processing engine splits the batch and runs multiple detection algorithms concurrently.

*   **Pipeline A: Deceased Beneficiary Detection**
    *   Cross-references Aadhaar IDs and exact/fuzzy name matches against the Civil Death Register.
*   **Pipeline B: Duplicate Identity (Transliteration-Aware)**
    *   Normalizes Gujarati names to a standard phonetic representation.
    *   Applies fuzzy matching algorithms (e.g., Double Metaphone, Levenshtein distance) to group highly similar names within the same scheme payload.
*   **Pipeline C: Cross-Scheme Duplication**
    *   Queries historical data and current batch to find identical Aadhaar/Bank combinations claiming mutually exclusive schemes.
*   **Pipeline D: Undrawn Funds / Middlemen Detection**
    *   Analyzes historical bank API responses. Flags accounts that receive funds but show zero withdrawal activity over long periods, or single bank accounts linked to multiple distinct beneficiaries (middlemen pattern).

### Phase 3: Risk Scoring and Aggregation
1.  For every flagged record, the engine calculates a composite Risk Score (0-100).
2.  The engine compiles the exact evidence strings based on which rules triggered.
3.  **Output:** Clean transactions proceed to the disbursement gateway. Flagged transactions are routed to the DFO's holding queue. Funds for flagged transactions are temporarily frozen.

## 2. Investigation and Resolution Workflow (Closed-Loop)

This workflow describes the human-in-the-loop process for handling flagged transactions.

1.  **Queue Generation:** The system populates the DFO Dashboard with newly flagged transactions, ordered by Risk Score.
2.  **DFO Triage:**
    *   DFO reviews the top priority item.
    *   *Decision Branch A (Clear False Positive):* DFO determines the flag is incorrect based on evidence. Clicks "Dismiss & Release Funds". System logs the action and releases payment.
    *   *Decision Branch B (Requires Verification):* DFO assigns the case to a Scheme Verifier.
3.  **Field Verification:**
    *   Scheme Verifier travels to the beneficiary location.
    *   Verifier logs into the mobile app, triggering GPS coordinate capture.
    *   Verifier conducts the audit, takes photos/notes, and selects a standardized outcome (e.g., "Confirmed Deceased", "Beneficiary Authentic", "Account belongs to agent").
    *   Report is synced back to the server.
4.  **Final Adjudication:**
    *   DFO reviews the Verifier's report.
    *   DFO makes final ruling: "Confirm Fraud" or "Release Funds".
    *   If fraud is confirmed, the beneficiary is blacklisted, and the audit trail is finalized for reporting.

## 3. System Configuration Workflow

1.  **State Admin Action:** The State DBT Admin identifies that the phonetic name matching is missing common Gujarati surname variations (e.g., "Patel" vs "Patil").
2.  **Dictionary Update:** Admin accesses the configuration dashboard and updates the custom phonetic dictionary.
3.  **Test Run:** Admin triggers a "dry run" against a historical dataset to observe the impact on false positive/true positive rates.
4.  **Deployment:** Admin commits the change. The new rules take effect on the next live data batch.
