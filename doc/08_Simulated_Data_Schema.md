# DBT Leakage Detection System: Simulated Data Schema & Analysis

This document details the schema of the provided datasets (`TS-PS4-1.csv` and `TS-PS4-2.csv`) and analyzes how their structure supports the required leakage detection rules for the hackathon.

## 1. Primary Transactions Dataset (`data/TS-PS4-1.csv`)

This file contains the historical and current disbursement records, acting as the primary payload for the anomaly detection engine.

### Data Dictionary
| Field | Type | Description | Observed Mock Data Examples |
| :--- | :--- | :--- | :--- |
| `beneficiary_id` | String | Unique system ID | `B100000`, `B100001` |
| `aadhaar` | Integer | 12-digit Aadhaar Number | `223005401501`, `130201276659` |
| `name` | String | Beneficiary Name (with transliteration variations) | `Suresh Patel`, `Suresh Ptl`, `Sureshbhai Patel` |
| `scheme` | String | Welfare Scheme Identifier | `PM-KISAN`, `Pension`, `Scholarship` |
| `district` | String | Gujarat District | `Surat`, `Bhavnagar`, `Rajkot`, `Ahmedabad` |
| `amount` | Integer | Disbursed Amount (₹) | `1000`, `2000`, `3000`, `5000` |
| `transaction_date`| Date | Date of fund transfer (YYYY-MM-DD) | `2023-07-29`, `2024-02-12` |
| `withdrawn` | Boolean | `1` if funds were withdrawn, `0` if sitting dormant | `0`, `1` |
| `status` | String | Gateway status of the transaction | `SUCCESS`, `FAILED` |

### Dataset Analysis (Leakage Coverage)
*   **Volume:** Contains 50,000+ transaction records, perfect for benchmarking the "10,000+ transactions in 30 seconds" requirement.
*   **Undrawn Funds Detection:** The `withdrawn` column explicitly flags dormant funds. If `withdrawn = 0` across multiple historical transactions for the same `beneficiary_id`, it signals a high probability of a middleman or an unaware beneficiary.
*   **Duplicate Identity (Transliterations):** The `name` column purposefully contains injected variations of standard Gujarati names (e.g., *Mahesh Shah*, *Maheshbhai Shah*, *Mahesh S.*). This perfectly allows the testing of Levenshtein/Fuzzy matching algorithms.
*   **Cross-Scheme Verification:** Beneficiaries can be grouped by `aadhaar` to check if they are claiming mutually exclusive benefits across `PM-KISAN`, `Pension`, and `Scholarship`.

---

## 2. Civil Death Register Dataset (`data/TS-PS4-2.csv`)

This file represents the official vital statistics registry. The ML engine will cross-reference transactions against this list to catch disbursements to deceased individuals.

### Data Dictionary
| Field | Type | Description | Observed Mock Data Examples |
| :--- | :--- | :--- | :--- |
| `aadhaar` | Integer | 12-digit Aadhaar Number | `893725541723`, `986322242576` |
| `name` | String | Deceased Individual's Name | `Mahesh Shah`, `Amit Joshi` |
| `death_date` | Date | Official Date of Death (YYYY-MM-DD) | `2025-11-29`, `2023-02-26` |

### Dataset Analysis (Leakage Coverage)
*   **Volume:** Contains 1,000 death records.
*   **Deceased Beneficiary Detection:** The core logic engine must execute a join operation between `TS-PS4-1.csv` and `TS-PS4-2.csv` on the `aadhaar` column.
*   **Temporal Logic:** The engine must compare the transaction's `transaction_date` against the registry's `death_date`. A flag should only be triggered if `transaction_date > death_date` (i.e., funds were sent *after* the person died).
