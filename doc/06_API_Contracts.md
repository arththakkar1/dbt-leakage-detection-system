# DBT Leakage Detection System: API Contracts

This document outlines the core RESTful API contracts between the Next.js Frontend, the Core Backend Orchestrator, and the Python/Flask ML Service.

## 1. Core Backend -> Python ML Service

### `POST /api/ml/v1/score-batch`
Sends a batch of new transactions to the ML engine for anomaly detection and risk scoring.

**Request Payload:**
```json
{
  "batch_id": "b_78910",
  "transactions": [
    {
      "transaction_id": "txn_1001",
      "beneficiary_id": "ben_554",
      "aadhaar_hash": "a1b2c3d4...",
      "name_english": "Ramesh Patel",
      "name_gujarati": "રમેશ પટેલ",
      "scheme_id": "sch_01",
      "bank_account": "1234567890",
      "ifsc": "SBIN0001234",
      "amount": 2500.00
    }
    // ... up to 10,000 records
  ]
}
```

**Response Payload (200 OK):**
```json
{
  "batch_id": "b_78910",
  "processing_time_ms": 14500,
  "total_processed": 10000,
  "flagged_transactions": [
    {
      "transaction_id": "txn_1001",
      "flag_type": "DUPLICATE_IDENTITY_TRANSLITERATION",
      "risk_score": 85,
      "evidence_summary": "Name 'Ramesh Patel' matches 'Rames Patill' on transaction txn_0992 with 92% Levenshtein phonetic similarity."
    }
  ]
}
```

## 2. Next.js Frontend -> Core Backend

### `GET /api/v1/investigations/queue`
Retrieves the prioritized queue of flagged transactions for the DFO Dashboard.

**Query Parameters:**
*   `district_id` (string)
*   `status` (string) - OPEN, INVESTIGATING
*   `limit` (int) - Pagination limit

**Response Payload (200 OK):**
```json
{
  "data": [
    {
      "case_id": "case_9001",
      "transaction_id": "txn_1001",
      "beneficiary_name": "Ramesh Patel",
      "scheme_name": "Old Age Pension",
      "flag_type": "DUPLICATE_IDENTITY_TRANSLITERATION",
      "risk_score": 85,
      "status": "OPEN",
      "created_at": "2026-05-04T10:00:00Z"
    }
  ],
  "pagination": {
    "total_records": 142,
    "current_page": 1,
    "total_pages": 15
  }
}
```

### `POST /api/v1/investigations/{case_id}/verify`
Submitted by the Scheme Verifier from the field via the mobile app.

**Request Payload:**
```json
{
  "verifier_id": "ver_005",
  "gps_lat": 23.0225,
  "gps_long": 72.5714,
  "timestamp": "2026-05-04T12:30:00Z",
  "finding": "CONFIRMED_FRAUD",
  "notes": "Beneficiary confirmed deceased 2 years ago by neighbors. Aadhaar being used by relative.",
  "photo_evidence_url": "https://storage.bucket.com/evidence/case_9001_photo.jpg"
}
```

**Response Payload (201 Created):**
```json
{
  "success": true,
  "message": "Verification report submitted and case updated.",
  "case_status": "RESOLVED_FRAUD"
}
```
