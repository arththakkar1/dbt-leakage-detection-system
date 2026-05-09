# DBT Leakage Detection System(WIP)

The Direct Benefit Transfer (DBT) Leakage Detection System is an advanced, real-time transaction monitoring and anomaly detection platform designed for the state of Gujarat. The core objective of the system is to identify, flag, and mitigate fraudulent or erroneous disbursements of welfare funds across multiple state-sponsored schemes before the next payment cycle occurs.

## Key Capabilities

- **Proactive Anomaly Detection:** Shifts operational focus from post-audit discovery to near real-time flagging of suspicious transactions.
- **High-Performance Processing:** Architected to ingest and evaluate over 10,000 transactions in under 30 seconds to support high-volume payment cycles via in-memory caching and parallel processing.
- **Explainable AI Framework:** Provides a quantifiable, deterministic Risk Score for every flagged transaction, accompanied by specific evidentiary citations (e.g., Levenshtein distance metrics for transliteration duplicates).
- **Advanced Machine Learning:** Utilizes an ensemble of models ranging from unsupervised Isolation Forests to Supervised Deep Learning Neural Networks (PyTorch & TensorFlow) to isolate complex fraud behaviors like dormant fund confiscation.
- **Closed-Loop Resolution:** Enables end-to-end tracking of flagged anomalies, from initial detection to field verification via GPS-stamped reporting by scheme verifiers.

## Technology Stack

The system utilizes a decoupled, microservices-inspired architecture:

- **Frontend (Next.js):** Provides role-based dashboards for District Finance Officers, Verifiers, and State Administrators with Server-Side Rendering for large data sets.
- **ML Engine (Python / Flask):** Encapsulates the core anomaly detection logic, utilizing libraries like FuzzyWuzzy for phonetic matching and Pandas for high-speed set operations against vital statistics registries.
- **Core Backend Orchestrator:** Language-agnostic middle-tier responsible for schema validation, high-speed database ingestion, and routing payloads to the ML engine.
- **Data Layer:** PostgreSQL for persistent, relational storage of transaction logs and audit trails, paired with Redis for low-latency in-memory lookups during batch processing.

## Documentation Navigation

Our documentation follows the Diataxis framework for clarity and structure. Please refer to the `doc/` directory for comprehensive system details:

### 1. Explanation (Understanding the System)

- [System Overview and Architecture](doc/explanation/system-overview-and-architecture.md)
- [Machine Learning Models and Features](doc/explanation/ml-models-and-features.md)
- [Frontend Architecture](doc/explanation/frontend-architecture.md)
- [Model Workflows](doc/explanation/model-workflows.md)
- [Problem Statement](doc/explanation/problem-statement.md)
- [Use Cases and Workflows](doc/explanation/use-cases-and-workflows.md)
- [Future Roadmap](doc/explanation/future-roadmap.md)

### 2. Tutorials (Learning)

- [Getting Started: Running the Simulation](doc/tutorials/getting-started.md)

### 3. Reference (Technical Details)

- [Data Schema and ER Diagram](doc/reference/data-schema-and-er.md)
- [Leakage Rules Engine](doc/reference/leakage-rules-engine.md)
- [API Contracts](doc/reference/api-contracts.md)

### 4. How-to Guides (Problem Solving)

- [Deploy and Scale](doc/how-to/deploy-and-scale.md)

## Getting Started

To test the system locally or explore the interactive Jupyter Notebooks detailing our machine learning pipeline:

Please follow the [Getting Started Guide](doc/tutorials/getting-started.md) to initialize the PyTorch/TensorFlow models, the Python ML Engine, and the Next.js frontend application.
