# DBT Leakage Detection System: Simulated Data Schema

To effectively test and demonstrate the system during the hackathon, we generate mock data representing the reality of DBT databases.

## 1. Beneficiary Database `beneficiaries_mock.csv`
Contains the demographic details of enrolled citizens.

| Field | Type | Description | Mock Example |
| :--- | :--- | :--- | :--- |
| `beneficiary_id` | String | Unique system ID | `ben_1009` |
| `aadhaar_hash` | String | SHA-256 hash of Aadhaar | `e3b0c442...` |
| `name_english` | String | Name in Latin script | `Kantibhai Desai` |
| `name_gujarati` | String | Name in Gujarati script | `કાંતિભાઈ દેસાઈ` |
| `dob` | Date | Date of Birth | `1965-04-12` |
| `district_id` | String | Standard district code | `GJ-01` (Ahmedabad) |
| `address` | String | Residential address | `12, M.G. Road...` |

## 2. Transaction Payload `transactions_batch.json`
Represents the array of disbursements fired during a payment cycle. This is the payload that needs to be processed in under 30 seconds.

```json
[
  {
    "transaction_id": "tx_99821",
    "beneficiary_id": "ben_1009",
    "scheme_id": "SCH_WIDOW_PENSION",
    "amount": 1250.00,
    "bank_account_no": "98765432101",
    "ifsc_code": "SBIN0004321",
    "timestamp": "2026-05-04T09:00:00Z"
  },
  {
    "transaction_id": "tx_99822",
    "beneficiary_id": "ben_4044",
    "scheme_id": "SCH_OLD_AGE",
    "amount": 2000.00,
    "bank_account_no": "11223344556",
    "ifsc_code": "HDFC0001234",
    "timestamp": "2026-05-04T09:00:01Z"
  }
]
```

## 3. Civil Death Register `death_register_mock.csv`
Simulates the state's vital statistics database. The ML engine cross-references against this.

| Field | Type | Description | Mock Example |
| :--- | :--- | :--- | :--- |
| `record_id` | String | Unique register ID | `dr_5502` |
| `aadhaar_hash` | String | SHA-256 hash (if available) | `e3b0c442...` |
| `name` | String | Full name on certificate | `Kantibhai Desai` |
| `date_of_death`| Date | Official Date of Death | `2025-11-20` |
| `district_id` | String | District of registration | `GJ-01` |

## Data Generation Strategy
To demonstrate the system's effectiveness:
1.  **Baseline Data:** Generate 9,500 clean, valid transactions.
2.  **Injected Fraud:** Purposely inject 500 fraudulent records that violate the 4 core rules (e.g., intentionally creating a transaction for a beneficiary whose Aadhaar hash is in the `death_register_mock.csv`).
3.  **Transliteration Variations:** Use python scripts to slightly alter English spellings of Gujarati names to test the Levenshtein distance engine.
