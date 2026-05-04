# Future Roadmap

> [!TIP]
> This document outlines the long-term vision and scalability plans for the DBT 
> Leakage Detection System beyond the initial prototype phase.

## Table of Contents
- [1. Phase 2: Advanced AI and UX (Months 1-6)](#1-phase-2-advanced-ai-and-ux-months-1-6)
- [2. Phase 3: Systemic Integrity (Months 6-12)](#2-phase-3-systemic-integrity-months-6-12)

## 1. Phase 2: Advanced AI and UX (Months 1-6)

### Graph Neural Networks (GNNs) for Cartel Detection
*   **Current:** Rule-based logic flags simple middlemen.
*   **Future:** Implement Graph Databases (Neo4j) and GNNs to map complex relationships and detect organized fraud rings.

### WhatsApp Integration for Verifiers
*   **Current:** Verifiers use a dedicated mobile web-app.
*   **Future:** Integrate WhatsApp Business API. Verifiers receive cases and submit GPS/photos directly via chat.

### Automated Bank API Integration
*   **Current:** Relies on simulated bank withdrawal data.
*   **Future:** Secure integration with Account Aggregator frameworks to pull real-time balances.

## 2. Phase 3: Systemic Integrity (Months 6-12)

### Blockchain for Immutable Audit Trails
*   **Current:** Audit logs in PostgreSQL.
*   **Future:** Write fraud case hashes and GPS data to a permissioned blockchain (Hyperledger Fabric) to prevent record tampering.

### Multi-Lingual NLP Support
*   **Current:** Focused on Gujarati-English transliteration.
*   **Future:** Expand NLP models for Hindi, Marathi, and other regional dialects.

### Predictive Disbursement Analytics
*   **Current:** Reactive anomaly detection.
*   **Future:** Predictive ML to forecast high-risk districts before disbursements begin.
