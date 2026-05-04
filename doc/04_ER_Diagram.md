# DBT Leakage Detection System: Data Models & ER Diagram

This document defines the underlying data structures required to support high-speed processing and robust audit trails.

## 1. Entity Relationship Diagram

```mermaid
erDiagram
    BENEFICIARY {
        string beneficiary_id PK
        string aadhaar_hash
        string full_name_english
        string full_name_gujarati
        string phonetic_hash
        string address
        string district_id
        date dob
    }

    SCHEME {
        string scheme_id PK
        string scheme_name
        string department
        boolean allows_multiple
    }

    TRANSACTION {
        string transaction_id PK
        string beneficiary_id FK
        string scheme_id FK
        string bank_account_no
        string ifsc_code
        decimal amount
        datetime timestamp
        string status "PENDING, CLEARED, FLAGGED, BLOCKED"
    }

    DEATH_REGISTER {
        string record_id PK
        string aadhaar_hash
        string full_name
        date date_of_death
        string death_certificate_no
    }

    FLAGGED_CASE {
        string case_id PK
        string transaction_id FK
        string flag_type "DECEASED, DUPLICATE, UNDRAWN, CROSS_SCHEME"
        int risk_score
        string evidence_summary
        string assigned_verifier_id FK
        string resolution_status "OPEN, INVESTIGATING, RESOLVED_FRAUD, RESOLVED_CLEARED"
    }

    VERIFICATION_REPORT {
        string report_id PK
        string case_id FK
        string verifier_id FK
        float gps_lat
        float gps_long
        datetime verification_time
        string finding_notes
        string attachment_url
    }

    USER {
        string user_id PK
        string role "DFO, VERIFIER, AUDITOR, ADMIN"
        string name
        string district_id
    }

    BENEFICIARY ||--o{ TRANSACTION : "receives"
    SCHEME ||--o{ TRANSACTION : "processes"
    TRANSACTION ||--o| FLAGGED_CASE : "generates"
    FLAGGED_CASE ||--o| VERIFICATION_REPORT : "results_in"
    USER ||--o{ FLAGGED_CASE : "manages"
    BENEFICIARY ||--o| DEATH_REGISTER : "matches against"
```

## 2. Key Data Entities

### 2.1 BENEFICIARY
Stores the core demographic data of individuals enrolled in any scheme.
*   **aadhaar_hash:** Aadhaar numbers must be encrypted or hashed at rest for security compliance.
*   **phonetic_hash:** A pre-calculated phonetic representation of the Gujarati name to drastically speed up transliteration-aware duplicate detection during real-time processing.

### 2.2 TRANSACTION
The primary high-volume table representing individual disbursement events.
*   Must be highly optimized for fast inserts and reads.
*   **status:** Tracks the lifecycle. Starts as PENDING, immediately moves to CLEARED if no rules trigger, or FLAGGED if anomalies are detected.

### 2.3 FLAGGED_CASE
The core entity for the DFO workflow. Extracted from raw transactions to create a structured investigation queue.
*   **risk_score:** An integer (0-100) representing the severity and confidence of the flag.
*   **evidence_summary:** A serialized JSON or structured string explaining exactly *why* the score was given, ensuring explainability.

### 2.4 VERIFICATION_REPORT
Stores the immutable ground-truth data collected by Verifiers.
*   **gps_lat / gps_long:** Critical fields to mathematically prove the Verifier was within a certain radius of the beneficiary's registered address.

## 3. Database Strategy for Performance
To meet the benchmark of processing 10,000+ transactions in under 30 seconds:
*   **In-Memory Caching:** The `DEATH_REGISTER` and recent `BENEFICIARY` phonetic hashes should be loaded into an in-memory datastore (like Redis) prior to batch processing to prevent disk I/O bottlenecks.
*   **Indexing:** Heavy indexing on `aadhaar_hash`, `phonetic_hash`, and `bank_account_no`.
*   **Partitioning:** The `TRANSACTION` table should be partitioned by date or month to maintain query speed as the dataset grows over years.
