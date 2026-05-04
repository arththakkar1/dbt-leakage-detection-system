# Getting Started: Running the Simulation

> [!NOTE]
> This tutorial will guide you through setting up the DBT Leakage Detection System locally,
> generating the simulated datasets, and running the core microservices.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Step 1: Clone and Setup](#step-1-clone-and-setup)
- [Step 2: Generate Mock Data](#step-2-generate-mock-data)
- [Step 3: Run the Python ML Engine](#step-3-run-the-python-ml-engine)
- [Step 4: Run the Next.js Frontend](#step-4-run-the-next-js-frontend)

## Prerequisites
*   Node.js (v18+)
*   Python (3.9+)
*   Git

## Step 1: Clone and Setup

First, clone the repository to your local machine:

```bash
git clone https://github.com/arththakkar1/dbt-leakage-detection-system.git
cd dbt-leakage-detection-system
```

## Step 2: Generate Mock Data

The repository contains a powerful data generator to simulate transactions and death registry records.

```bash
# Install required Python libraries
pip install faker pandas

# Run the data generation script
python generate_mock_data.py
```

> [!TIP]
> This will create two files in the `data/` directory: `TS-PS4-1.csv` (Transactions) 
> and `TS-PS4-2.csv` (Death Register), complete with injected anomalies for testing!

## Step 3: Run the Python ML Engine

The core logic resides in the Flask application.

```bash
cd ml-engine/
pip install -r requirements.txt
python app.py
```

The ML service will now be listening on `http://localhost:5000`.

## Step 4: Run the Next.js Frontend

Open a new terminal window to start the interactive dashboard.

```bash
cd frontend/
npm install
npm run dev
```

Navigate to `http://localhost:3000` in your browser. You can now simulate a batch upload and watch the anomaly detection rules in action!
