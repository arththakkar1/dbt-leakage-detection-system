# Use Cases and Workflows

> [!NOTE]
> This document outlines the functional requirements, use cases for each user role,
> and the operational workflows for the DBT Leakage Detection System.

## Table of Contents
- [1. District Finance Officer (DFO) Use Cases](#1-district-finance-officer-dfo-use-cases)
- [2. Scheme Verifier Use Cases](#2-scheme-verifier-use-cases)
- [3. Core Data Processing Workflow](#3-core-data-processing-workflow)
- [4. Investigation and Resolution Workflow](#4-investigation-and-resolution-workflow)

## 1. District Finance Officer (DFO) Use Cases

### UC-DFO-01: View Prioritized Investigation Queue
*   **Trigger:** DFO logs into the dashboard.
*   **Main Flow:** System presents flagged transactions, strictly prioritized by Risk Score (Descending).

### UC-DFO-02: Review Transaction Evidence
*   **Trigger:** DFO selects a specific flagged transaction.
*   **Main Flow:** System displays detailed beneficiary data and explainable evidence 
    (e.g., "Aadhaar XXXX matched Civil Death Register entry #88902 with 98% confidence").

### UC-DFO-03: Assign Case to Field Investigator
*   **Trigger:** DFO decides field verification is necessary.
*   **Main Flow:** DFO clicks "Assign for Verification" and selects a Verifier. System notifies the Verifier.

## 2. Scheme Verifier Use Cases

### UC-SV-01: Submit GPS-Stamped Verification Report
*   **Trigger:** Verifier arrives at the beneficiary location.
*   **Main Flow:**
    1. Verifier clicks "Start Verification".
    2. System immediately captures latitude, longitude, and timestamp.
    3. Verifier fills out a dynamic form and submits.
*   **Exception:** Poor GPS accuracy warns the Verifier and flags submission for secondary review.

## 3. Core Data Processing Workflow

This high-speed backend process handles 10,000+ transactions in under 30 seconds.

### Phase 1: Data Ingestion
1.  System receives DBT payment records.
2.  Loads latest snapshot of Civil Death Register and Aadhaar mapping into Redis.

### Phase 2: Parallel Anomaly Detection
*   **Pipeline A:** Deceased Beneficiary Detection (Aadhaar cross-reference).
*   **Pipeline B:** Duplicate Identity (Double Metaphone, Levenshtein distance).
*   **Pipeline C:** Cross-Scheme Duplication.
*   **Pipeline D:** Undrawn Funds / Middlemen Detection (Account analysis).

### Phase 3: Risk Scoring
*   Engine calculates a composite Risk Score (0-100) and compiles evidence strings.

## 4. Investigation and Resolution Workflow

1.  **Queue Generation:** DFO Dashboard populated with flagged transactions.
2.  **DFO Triage:** DFO reviews and assigns to Verifier or dismisses.
3.  **Field Verification:** Verifier conducts audit via mobile app with GPS tracking.
4.  **Final Adjudication:** DFO reviews Verifier report and makes final ruling ("Confirm Fraud" or "Release Funds").
