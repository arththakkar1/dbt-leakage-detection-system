# DBT Leakage Detection System: Future Roadmap

While the hackathon prototype covers the critical core requirements, a state-wide deployment of this system offers massive potential for future enhancements. Here is the roadmap for Phase 2 and Phase 3.

## Phase 2: Advanced AI and UX Enhancements (Months 1-6)

### 1. Graph Neural Networks (GNNs) for Cartel Detection
*   **Current State:** Rule-based logic flags simple middlemen (multiple IDs to 1 bank account).
*   **Future State:** Implement Graph Databases (like Neo4j) and GNNs to map complex relationships. If Beneficiary A shares an address with B, and B shares a phone number with C, and C's funds go to D's bank account, the system can detect highly organized fraud rings that bypass simple rules.

### 2. WhatsApp Integration for Verifiers
*   **Current State:** Verifiers use a dedicated mobile web-app.
*   **Future State:** Integrate with the WhatsApp Business API. Verifiers receive cases as WhatsApp messages, and they can reply with their location pin (GPS stamp) and photos directly in the chat, dramatically reducing the friction and training required for on-ground staff.

### 3. Automated Bank API Integration (Open Banking)
*   **Current State:** "Undrawn Funds" relies on simulated data.
*   **Future State:** Securely integrate with banking APIs (via NPCI or Account Aggregator framework) to pull real-time account balances and last-withdrawal dates automatically, verifying dormancy without manual intervention.

## Phase 3: Systemic Integrity and Scaling (Months 6-12)

### 4. Blockchain for Immutable Audit Trails
*   **Current State:** Audit logs are stored in PostgreSQL.
*   **Future State:** Write the hash of every "Resolved Fraud" case and its corresponding evidence/GPS data to a permissioned blockchain (like Hyperledger Fabric). This ensures that once fraud is detected and verified, the record cannot be altered or deleted by corrupt officials at any level.

### 5. Multi-Lingual NLP Support
*   **Current State:** Focused on Gujarati to English transliteration.
*   **Future State:** Expand the NLP models to handle Hindi, Marathi, and other regional dialects seamlessly, allowing the system to be licensed and deployed by other Indian states.

### 6. Predictive Disbursement Analytics
*   **Current State:** Reactive and near-real-time anomaly detection.
*   **Future State:** Train predictive ML models on historical leakage data to forecast *which* districts or specific schemes are most likely to experience fraud in the upcoming quarter, allowing the State Admin to preemptively assign more Verification resources to high-risk areas before disbursements even begin.
