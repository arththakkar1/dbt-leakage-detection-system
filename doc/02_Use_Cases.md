# DBT Leakage Detection System: Detailed Use Cases

This document outlines the exhaustive functional requirements mapped to specific use cases for each user role within the DBT Leakage Detection System.

## 1. Use Cases for District Finance Officer (DFO)

### UC-DFO-01: View Prioritized Investigation Queue
*   **Trigger:** DFO logs into the dashboard.
*   **Pre-condition:** The anomaly detection engine has processed a batch of transactions.
*   **Main Flow:**
    1. The system presents a tabular or card-based view of flagged transactions.
    2. The list is strictly prioritized by the calculated Risk Score (Descending order).
    3. Each entry displays: Beneficiary Name, Scheme, Flag Type, Risk Score, and a brief summary of evidence.
*   **Post-condition:** DFO understands which cases require immediate attention.

### UC-DFO-02: Review Transaction Evidence
*   **Trigger:** DFO selects a specific flagged transaction from the queue.
*   **Main Flow:**
    1. System displays detailed beneficiary data (demographics, bank details).
    2. System displays the exact leakage rule triggered (e.g., "Deceased Detection").
    3. System provides explainable evidence (e.g., "Aadhaar XXXX-XXXX-1234 matched against Civil Death Register entry #88902 with 98% confidence. Date of death precedes transfer date.").
*   **Alternative Flow:** If transliteration duplicate is flagged, the system displays side-by-side comparisons of the two suspected duplicate profiles highlighting phonetic similarities.

### UC-DFO-03: Assign Case to Field Investigator
*   **Trigger:** After reviewing evidence, DFO decides field verification is necessary.
*   **Main Flow:**
    1. DFO clicks "Assign for Verification".
    2. System lists available Scheme Verifiers in the corresponding sub-district/village.
    3. DFO selects a Verifier and adds optional remarks.
    4. System updates the transaction status to "Under Investigation" and notifies the Verifier.

### UC-DFO-04: Export Structured Audit Report
*   **Trigger:** DFO needs to submit weekly/monthly reports to the State Finance Department.
*   **Main Flow:**
    1. DFO selects date range and schemes.
    2. System compiles a structured report prioritizing high-risk flags, total funds frozen, and cases resolved.
    3. The report is exported in standard formats (CSV, PDF) adhering to state reporting standards.

## 2. Use Cases for Scheme Verifier

### UC-SV-01: Receive and Review Case Assignment
*   **Trigger:** Verifier receives a push notification or SMS about a new assignment.
*   **Main Flow:**
    1. Verifier opens the mobile application.
    2. System displays assigned cases prioritized by urgency or proximity.
    3. Verifier views the beneficiary address, suspected flag (e.g., "Suspected Undrawn Funds"), and DFO remarks.

### UC-SV-02: Submit GPS-Stamped Verification Report
*   **Trigger:** Verifier arrives at the beneficiary location.
*   **Main Flow:**
    1. Verifier clicks "Start Verification".
    2. System immediately captures latitude, longitude, and timestamp.
    3. Verifier fills out a dynamic form based on the flag type (e.g., For "Deceased" flag, the form prompts: "Is the beneficiary alive?", "Upload photo of beneficiary or death certificate").
    4. Verifier submits the form.
*   **Exception:** If GPS accuracy is poor or location is too far from the registered address, the system warns the Verifier and flags the submission for secondary DFO review.

## 3. Use Cases for Audit Team Member

### UC-AUD-01: Query Cross-Scheme Duplicates
*   **Trigger:** Auditor conducts periodic checks for systemic fraud.
*   **Main Flow:**
    1. Auditor accesses the advanced query interface.
    2. Selects multiple schemes (e.g., Widow Pension AND Disability Pension).
    3. System runs cross-reference algorithms and returns records where identities overlap suspiciously across schemes not meant to be combined.

### UC-AUD-02: Generate Compliance Summary
*   **Trigger:** Auditor prepares quarterly governance reports.
*   **Main Flow:**
    1. Auditor requests a compliance aggregate.
    2. System calculates metrics: Total transactions, Number of flags, False positive rate (flags overturned by Verifiers), and Total monetary value of leakage prevented.

## 4. Use Cases for State DBT Admin

### UC-ADM-01: View State-Level Risk Heatmap
*   **Trigger:** Admin monitors overall state health.
*   **Main Flow:**
    1. System renders a geographical map of Gujarat.
    2. Districts are color-coded based on the density and severity of flagged transactions.
    3. Admin filters by Leakage Type (e.g., showing only "Middlemen Account" flags) to identify geographical clusters of specific fraud types.

### UC-ADM-02: Configure Leakage Pattern Rules
*   **Trigger:** A new type of fraud is discovered, or existing rules produce too many false positives.
*   **Main Flow:**
    1. Admin accesses the Rule Engine Configuration.
    2. Adjusts parameters (e.g., changing the Gujarati transliteration Levenshtein distance threshold from 2 to 3, or updating the threshold for "Undrawn Funds" from 90 days to 120 days).
    3. System saves configuration and applies it to the next processing batch.
