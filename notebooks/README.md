# Machine Learning Notebooks

This directory contains Jupyter notebooks that demonstrate the proof-of-code implementation for the various machine learning models used in the DBT Leakage Detection System.

## Notebook Sequence
- **`01_isolation_forest_dormant_funds.ipynb`** (Model A): Detects dormant funds anomalies using Scikit-Learn's `IsolationForest` on behavioral transaction features.
- **`02_fuzzy_transliteration_matching.ipynb`** (Model B): Detects duplicate identities resulting from spelling transliterations using `FuzzyWuzzy` and Levenshtein distance.
- **`03_prescriptive_ai_suggestions.ipynb`** (Model C): Contains deterministic data joins (simulating high-speed heuristics) and an AI simulator to generate prescriptive instructions based on flagged anomalies.
- **`04_supervised_fraud_classification.ipynb`** (Model D): A comprehensive supervised learning pipeline comparing traditional models (Scikit-Learn) with deep neural networks (PyTorch and TensorFlow) trained on synthesized labels.
