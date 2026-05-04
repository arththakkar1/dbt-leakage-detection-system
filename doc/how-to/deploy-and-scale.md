# Deploy and Scale

> [!TIP]
> This guide explains how to deploy the system to hit the performance benchmark 
> of processing **10,000+ transactions in under 30 seconds**.

## Table of Contents
- [1. Containerization Strategy](#1-containerization-strategy)
- [2. Hitting the 10k/30s Benchmark](#2-hitting-the-10k30s-benchmark)
- [3. Kubernetes Production Deployment](#3-kubernetes-production-deployment)

## 1. Containerization Strategy

The entire application suite is containerized using **Docker**:
*   **Frontend Container:** Runs the Next.js production build.
*   **Backend Orchestrator Container:** Runs the core API logic.
*   **ML Engine Container:** Runs the Python Flask application.
*   **Database Container:** PostgreSQL for persistent storage.
*   **Cache Container:** Redis for in-memory data operations.

## 2. Hitting the 10k/30s Benchmark

Processing 333+ complex transactions per second requires specific optimizations:

### Redis In-Memory Caching (Crucial)
Checking a relational DB 10,000 times for Death Register matches will fail the time benchmark.
*   **Solution:** Before a transaction batch is processed, the system loads the Death Register Aadhaar Hashes and Active Beneficiary Phonetic Hashes into **Redis**. 
*   The Python ML engine performs its set intersections entirely in RAM (milliseconds).

### Parallel Processing (Python Multiprocessing)
*   **Solution:** Upon receiving the payload, Flask splits the array into chunks (e.g., 4 chunks of 2,500). It utilizes Python's `multiprocessing` to run the 4 rule pipelines simultaneously across multiple CPU cores.

### Database Batch Inserts
*   **Solution:** The system uses bulk `INSERT` and `UPDATE` statements to write the final transaction statuses back to PostgreSQL in large blocks, avoiding connection pool exhaustion.

## 3. Kubernetes Production Deployment

For state-level deployment, containers should be orchestrated using **Kubernetes (K8s)** on a secure government cloud (e.g., AWS GovCloud, NIC cloud).

*   **Auto-scaling:** The Python ML Engine is configured with a Horizontal Pod Autoscaler (HPA). If CPU spikes during a payment cycle, K8s automatically spins up more Flask pods.
*   **Load Balancing:** An Ingress Controller distributes incoming batches evenly.
*   **Data Security:** All inter-pod communication is encrypted, and databases are deployed in isolated private subnets.
