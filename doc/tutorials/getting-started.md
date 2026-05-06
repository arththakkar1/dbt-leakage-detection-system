# Getting Started: Running the Simulation

> [!NOTE]
> This tutorial will guide you through setting up the DBT Leakage Detection System locally,
> generating the simulated datasets, and running the core microservices.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Step 1: Clone and Setup](#step-1-clone-and-setup)
- [Step 2: Run the Python ML Engine](#step-2-run-the-python-ml-engine)
- [Step 3: Run the Next.js Frontend](#step-3-run-the-next-js-frontend)

## Prerequisites
*   Node.js (v18+)
*   Python (3.9+)
*   Git
*   Python Data Science Stack (`pip install pandas scikit-learn torch tensorflow fuzzywuzzy matplotlib seaborn`)

## Step 1: Clone and Setup

First, clone the repository to your local machine:

```bash
git clone https://github.com/arththakkar1/dbt-leakage-detection-system.git
cd dbt-leakage-detection-system
```

## Step 2: Run the Python ML Engine

The core logic resides in the Flask application.

```bash
cd ml-engine/
pip install -r requirements.txt
python app.py
```

The ML service will now be listening on `http://localhost:5000`.

## Step 3: Explore the Machine Learning Pipeline (Jupyter Notebooks)

Before running the full system, you can interact directly with the core algorithms using our executable Jupyter Notebooks. These notebooks ingest the real datasets (`data/TS-PS4-1.csv`) and visualize the detection logic.

```bash
cd ../notebooks/
# Launch Jupyter Lab or open the .ipynb files in VSCode
jupyter lab
```

You will find notebooks detailing:
1.  Isolation Forest anomaly detection (Dormant Funds).
2.  Levenshtein Distance string matching (Duplicate Identities).
3.  Supervised Neural Networks (PyTorch/TensorFlow).

## Step 4: Run the Next.js Frontend

Open a new terminal window to start the interactive dashboard.

```bash
cd ../frontend/
npm install
npm run dev
```

Navigate to `http://localhost:3000` in your browser. You can now simulate a batch upload and watch the anomaly detection rules in action!
