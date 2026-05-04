# DBT Leakage Detection System: Deployment and Scaling Guide

To meet the strict performance requirement of processing **10,000+ transactions in under 30 seconds**, the system architecture is designed for horizontal scalability and rapid data retrieval.

## 1. Containerization Strategy

The entire application suite is containerized using **Docker** to ensure environment consistency from local development to production.

*   **Frontend Container:** Runs the Next.js application (optimized production build).
*   **Backend Orchestrator Container:** Runs the core API logic (Node.js/Go).
*   **ML Engine Container:** Runs the Python Flask application, pre-loaded with necessary data science libraries (Pandas, FuzzyWuzzy).
*   **Database Container:** PostgreSQL for persistent storage.
*   **Cache Container:** Redis for in-memory data operations.

## 2. Meeting the 10k/30s Benchmark

Processing 333+ complex transactions per second requires specific optimization strategies:

### A. Redis In-Memory Caching (Crucial)
Checking a relational database 10,000 times for Death Register matches will fail the time benchmark due to disk I/O.
*   **Solution:** Before a transaction batch is processed, the system loads the necessary lookup tables (Death Register Aadhaar Hashes, Active Beneficiary Phonetic Hashes) into **Redis**.
*   The Python ML engine performs its set intersections and fuzzy matches entirely in RAM, operating in milliseconds.

### B. Parallel Processing (Python Multiprocessing)
The Python Flask service must not process the 10,000 array sequentially.
*   **Solution:** Upon receiving the payload, Flask splits the array into chunks (e.g., 4 chunks of 2,500).
*   It utilizes Python's `multiprocessing` or `concurrent.futures` to run the 4 rule pipelines simultaneously across multiple CPU cores.

### C. Database Batch Inserts
Writing 10,000 status updates individually to PostgreSQL will cause connection pool exhaustion and delays.
*   **Solution:** The system uses bulk `INSERT` and `UPDATE` statements to write the final transaction statuses and generated 'Flagged Cases' back to the database in large blocks.

## 3. Production Deployment Architecture (Kubernetes)

For state-level deployment, the Docker containers should be orchestrated using **Kubernetes (K8s)** on a secure government cloud infrastructure (e.g., AWS GovCloud, NIC cloud).

*   **Auto-scaling:** The Python ML Engine deployment is configured with a Horizontal Pod Autoscaler (HPA). If CPU utilization spikes during a massive payment cycle (e.g., 100,000 transactions), K8s automatically spins up more Flask pods to handle the load.
*   **Load Balancing:** An Ingress Controller distributes the incoming batches from the disbursement gateway evenly across the available Backend/ML pods.
*   **Data Security:** All inter-pod communication is encrypted, and databases are deployed in isolated private subnets, strictly adhering to government data privacy regulations.
