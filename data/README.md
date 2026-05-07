# Data Directory

This directory contains the datasets utilized by the Direct Benefit Transfer (DBT) Leakage Detection System. It includes simulated transactional data and civil vital statistics registries required for model training and simulation.

## Files
- **`TS-PS4-1.csv`**: Primary simulated transactional dataset containing fund disbursements. Includes fields like `aadhaar`, `beneficiary_id`, `amount`, `withdrawn`, etc.
- **`TS-PS4-2.csv`**: Civil registry dataset representing vital statistics (e.g., official death records). Includes `aadhaar` and `death_date` for identifying post-mortem disbursements.

*Note: In a production environment, this directory might be excluded from version control for security/size reasons, and datasets would be fetched from a secure database or data lake.*
