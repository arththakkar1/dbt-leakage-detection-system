# Data Directory

> [!IMPORTANT]
> This directory contains the real-world transactional datasets used by the DBT Leakage Detection System for machine learning model training, evaluation, and proof-of-concept demonstration. These files are the foundation of every anomaly detection algorithm in the `notebooks/` directory.

## Table of Contents
1. [Overview](#1-overview)
2. [Dataset Descriptions](#2-dataset-descriptions)
3. [Data Statistics](#3-data-statistics)
4. [Schema Reference](#4-schema-reference)
5. [How the Data is Used](#5-how-the-data-is-used)
6. [Data Privacy and Production Notes](#6-data-privacy-and-production-notes)

---

## 1. Overview

The datasets in this directory simulate the real-world data environment of a Gujarat state welfare disbursement system. The two CSV files represent the two core data sources that the leakage detection system must reconcile in real-time:

1. **The Payment Ledger (`TS-PS4-1.csv`):** Every disbursement made to every welfare beneficiary.
2. **The Civil Death Registry (`TS-PS4-2.csv`):** The official government record of deceased individuals.

The critical insight is that these two databases are **completely siloed** in legacy government systems. Joining them on `aadhaar` and checking if `transaction_date > death_date` is exactly what Model C (Deterministic Heuristics) does.

---

## 2. Dataset Descriptions

### `TS-PS4-1.csv` — Primary Transaction Dataset
The **primary, high-volume dataset** powering most of the ML pipeline. Contains 50,000 individual disbursement records across multiple welfare schemes and districts in Gujarat.

**Use in the System:**
- **Model A (Isolation Forest):** Aggregated by `beneficiary_id` to engineer behavioral features (`withdrawal_rate`, `total_amount`) for dormant fund anomaly detection.
- **Model B (Fuzzy Transliteration):** The `name` column is extracted, NLP-cleaned, and compared pairwise within districts using Levenshtein distance.
- **Model E (Supervised Classification):** Features are labeled via deterministic heuristics and used as training data for PyTorch/TensorFlow neural networks.

### `TS-PS4-2.csv` — Civil Death Registry
Represents the **State Civil Death Register**. In production, this would be a live API feed from the Registrar General of India (RGI).

**Use in the System:**
- **Model C (Deceased Detection):** Joined with `TS-PS4-1.csv` on `aadhaar`. Any transaction where `transaction_date > death_date` is flagged as `DECEASED_DISBURSEMENT` with a risk score of 100 (Critical).
- **Model D (Prescriptive AI):** `death_date` and `days_post_mortem` are injected as context into the LLM prompt.

---

## 3. Data Statistics

| Metric | `TS-PS4-1.csv` | `TS-PS4-2.csv` |
|---|---|---|
| **Total Records** | 50,000 | ~500 |
| **File Size** | ~3.8 MB | ~40 KB |
| **Key Join Column** | `aadhaar` | `aadhaar` |
| **Withdrawal Rate** | ~50% (uniform) | N/A |
| **Amount Range** | ₹1,000 – ₹5,000 | N/A |

---

## 4. Schema Reference

### `TS-PS4-1.csv` — Column Schema

| Column | Type | Description | Example |
|---|---|---|---|
| `beneficiary_id` | `string` | Unique beneficiary identifier | `BEN_00001` |
| `aadhaar` | `string` | 12-digit Aadhaar number | `123456789012` |
| `name` | `string` | Full name (Gujarati transliteration) | `Suresh Patel` |
| `scheme` | `string` | Welfare scheme name | `PM_KISAN` |
| `district` | `string` | District in Gujarat | `Surat` |
| `amount` | `float` | Disbursement in Rupees (₹) | `3000.0` |
| `transaction_date` | `string` (ISO 8601) | Fund transfer date | `2024-03-15` |
| `withdrawn` | `int` (0/1) | `1` = funds withdrawn, `0` = dormant | `0` |

### `TS-PS4-2.csv` — Column Schema

| Column | Type | Description | Example |
|---|---|---|---|
| `aadhaar` | `string` | Aadhaar linking to transaction dataset | `123456789012` |
| `name` | `string` | Deceased individual's name | `Ramesh Desai` |
| `death_date` | `string` (ISO 8601) | Official date of death | `2023-11-20` |

> [!WARNING]
> Always cast both `aadhaar` columns to `str` and strip whitespace before merging. A silent type mismatch results in zero matched rows — a false "no fraud detected" signal. This is handled explicitly in `notebooks/03_prescriptive_ai_suggestions.ipynb`.

---

## 5. How the Data is Used

All notebooks read from this directory using a relative path:

```python
df = pd.read_csv('../data/TS-PS4-1.csv')
deaths = pd.read_csv('../data/TS-PS4-2.csv')
```

The standard preprocessing pipeline applied before any model runs:

```python
# 1. Null handling
df.dropna(subset=['amount', 'withdrawn', 'beneficiary_id'], inplace=True)

# 2. Type casting
df['amount'] = pd.to_numeric(df['amount'], errors='coerce').fillna(0)
df['withdrawn'] = df['withdrawn'].astype(int)

# 3. Safe datetime parsing
df['transaction_date'] = pd.to_datetime(df['transaction_date'], errors='coerce')
df.dropna(subset=['transaction_date'], inplace=True)

# 4. Key alignment before join
df['aadhaar'] = df['aadhaar'].astype(str).str.strip()
deaths['aadhaar'] = deaths['aadhaar'].astype(str).str.strip()
```

---

## 6. Data Privacy and Production Notes

> [!CAUTION]
> This directory contains **simulated** data generated for demonstration purposes. No real beneficiary Aadhaar numbers or personal details are included.

In a **production deployment**, this `data/` directory would be managed as follows:

- **Excluded from Version Control:** `data/` would be added to `.gitignore` to prevent real citizen data from being committed.
- **Secure Data Lake:** Production datasets stored in encrypted object storage (AWS S3 with KMS, or NIC government data store).
- **Access Control:** Read access granted exclusively to the ML Engine service account via IAM roles.
- **Data Masking:** Aadhaar numbers displayed in the UI with only the last 4 digits visible (`XXXX-XXXX-9012`), conforming to UIDAI privacy regulations.
- **Audit Logging:** Every read operation against production data logged to an immutable audit trail.
