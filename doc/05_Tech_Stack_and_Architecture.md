# DBT Leakage Detection System: Technology Stack and Architecture

This document outlines the proposed technology stack for the DBT Leakage Detection System, focusing on a decoupled, service-oriented architecture to ensure high performance, maintainability, and scalability.

## 1. Overall Architecture Strategy

The system is designed with a microservices-inspired architecture, separating the user interface, the high-speed data processing backend, and the specialized machine learning anomaly detection engine. This separation of concerns allows each component to scale independently, which is crucial for processing 10,000+ transactions in under 30 seconds while simultaneously serving a responsive dashboard to administrators.

## 2. Frontend Application: Next.js

The user interface for all roles (District Finance Officer, Verifier, Auditor, State Admin) is built using **Next.js**, a powerful React framework.

### Why Next.js?
*   **Role-Based Dashboards:** Next.js allows for seamless creation of distinct, secure routes for different user roles.
*   **Server-Side Rendering (SSR) & Static Site Generation (SSG):** Ensures dashboards load instantly, critical for DFOs reviewing large tables of flagged transactions.
*   **API Routes:** Next.js can securely handle lightweight middle-tier routing, protecting backend endpoints and managing authentication state.
*   **Responsive Design:** Easily integrates with modern CSS frameworks (e.g., Tailwind CSS) to provide a seamless experience on desktop (for DFOs/Admins) and mobile devices (for Verifiers in the field).

### Frontend Responsibilities:
*   Rendering the prioritized investigation queue.
*   Displaying the state-level risk heatmap using mapping libraries (e.g., React-Leaflet or Mapbox).
*   Handling the GPS-stamped form submissions from Scheme Verifiers.

## 3. Anomaly Detection & ML Engine: Python (Flask)

The core intelligence of the system—the leakage detection algorithms and rule engines—is encapsulated in a dedicated Python microservice exposed via **Flask**.

### Why Python and Flask?
*   **Data Science Ecosystem:** Python is the industry standard for data manipulation and machine learning (Pandas, NumPy, Scikit-learn).
*   **Lightweight API:** Flask provides a highly performant, lightweight framework to expose the detection models as RESTful APIs without unnecessary overhead.
*   **Specialized Libraries:** Python provides robust libraries for string manipulation and phonetic matching, crucial for handling Gujarati transliterations.

### Model and Algorithm Specifics
*   **Fuzzy Name Matching:** Utilizing libraries like `FuzzyWuzzy` (Levenshtein distance) or custom implementations of the `Double Metaphone` algorithm tailored for Indian phonetics to catch duplicate identities across transliterations.
*   **Deceased Detection:** High-speed set operations (using Pandas or Redis integrations) to cross-reference Aadhaar hashes against the Civil Death Register.
*   **Behavioral Anomaly Models:** For detecting "Undrawn Funds" or "Middlemen Accounts," the Python service can utilize statistical anomaly detection (e.g., Isolation Forests) trained on historical withdrawal patterns, moving beyond simple hardcoded rules.

### ML Service Workflow
1.  The backend service sends a batch of standardized transaction data to the Flask API.
2.  The Flask service runs the data through the 4 core detection pipelines in parallel.
3.  The service calculates the Risk Score and returns a JSON payload containing flagged transactions and their associated evidence strings.

## 4. Core Backend Services: Decoupled Architecture

While the frontend is firmly set in Next.js and the ML engine in Python/Flask, the primary backend orchestrator is designed to be language-agnostic. This core backend (which could be implemented in Node.js, Go, or Java) acts as the central nervous system.

### Core Backend Responsibilities:
*   **High-Speed Ingestion:** Receiving the simulated 10,000+ transaction payload and validating the schema.
*   **Orchestration:** Routing the validated data to the Python/Flask ML service for scoring.
*   **Database Management:** Handling all CRUD operations for the core database (PostgreSQL/MySQL), storing transactions, updating flag statuses, and saving verification reports.
*   **Authentication & Authorization:** Managing secure access tokens for the Next.js frontend to ensure data privacy.
*   **Notifications:** Triggering SMS or push notifications to Verifiers when new cases are assigned.

This decoupled structure ensures that the computationally heavy ML processing in Python does not block the concurrent API requests coming from the Next.js frontend, ensuring a smooth experience for all users even during peak transaction processing times.
