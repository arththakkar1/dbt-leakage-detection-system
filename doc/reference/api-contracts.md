# API Contracts

> [!NOTE]
> Outlines the core RESTful API contracts between the Next.js Frontend, 
> the Core Backend Orchestrator, and the Python/Flask ML Service.

## Table of Contents
- [1. Backend to ML Service](#1-backend-to-ml-service)
- [2. Frontend to Backend](#2-frontend-to-backend)

## 1. Backend to ML Service

### `POST /api/ml/v1/score-batch`
Sends a batch of new transactions to the ML engine for anomaly detection.

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
      "amount": 2500.00
    }
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
      "evidence_summary": "Name 'Ramesh Patel' matches 'Rames Patill' with 92% similarity."
    }
  ]
}
```

## 2. Frontend to Backend

### `GET /api/v1/investigations/queue`
Retrieves the prioritized queue of flagged transactions for the DFO Dashboard.

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
  ]
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
  "finding": "CONFIRMED_FRAUD"
}
```
