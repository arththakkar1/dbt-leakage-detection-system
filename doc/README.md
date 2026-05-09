# Documentation

> [!NOTE]
> This directory is the single source of truth for all technical and conceptual documentation for the DBT Leakage Detection System. It is organized according to the **Diátaxis documentation framework**, a principled approach that separates documentation by the distinct needs it serves.

## Table of Contents

1. [What is Diátaxis?](#1-what-is-diataxis)
2. [Directory Structure](#2-directory-structure)
3. [Folder Descriptions](#3-folder-descriptions)
   - [explanation/](#31-explanation)
   - [reference/](#32-reference)
   - [tutorials/](#33-tutorials)
   - [how-to/](#34-how-to)
4. [Document Index](#4-document-index)
5. [Documentation Standards](#5-documentation-standards)

---

## 1. What is Diátaxis?

Diátaxis is a systematic framework for structuring technical documentation. It identifies four distinct user needs and assigns each a dedicated documentation type:

| Mode | Answers the Question | Oriented Towards | Practical / Theoretical |
|---|---|---|---|
| **Explanation** | Why does this work this way? | Understanding | Theoretical |
| **Reference** | What is the exact specification of X? | Information lookup | Theoretical |
| **Tutorial** | How do I get started from zero? | Learning | Practical |
| **How-to Guide** | How do I accomplish a specific goal? | Problem-solving | Practical |

By separating documentation into these four quadrants, this project ensures that a first-time contributor can find a **Tutorial** to get started, a senior engineer can look up a **Reference** document without wading through explanatory prose, and a product manager can read an **Explanation** document to understand system design decisions.

---

## 2. Directory Structure

```
doc/
├── explanation/                          # Theoretical understanding (the "Why")
│   ├── problem-statement.md              # Root cause analysis of DBT leakage in India
│   ├── system-overview-and-architecture.md # High-level system design and microservices
│   ├── ml-models-and-features.md        # Deep dive into each ML model and its features
│   ├── frontend-architecture.md         # Next.js routes, proxy.ts, and component design
│   ├── model-workflows.md               # End-to-end data flow for each detection scenario
│   ├── use-cases-and-workflows.md       # User-centered workflows for DFO, Verifier, Admin
│   └── future-roadmap.md               # Planned features and long-term system evolution
│
├── reference/                           # Technical specifications (the "What")
│   ├── data-schema-and-er.md            # PostgreSQL schema, entity relationships, field types
│   ├── leakage-rules-engine.md          # Formal definition of every detection rule and risk score math
│   └── api-contracts.md                 # Endpoint-by-endpoint API contract documentation
│
├── tutorials/                           # Getting-started guides (the "How to begin")
│   └── getting-started.md               # Step-by-step setup for first-time contributors
│
└── how-to/                              # Goal-oriented guides (the "How to do X")
    └── deploy-and-scale.md              # Instructions for deploying to cloud infrastructure
```

---

## 3. Folder Descriptions

### 3.1 `explanation/`

The `explanation/` folder contains documents focused on **building deep understanding** of the system. These documents are not guides — they do not instruct the reader to do anything. Instead, they explain the design decisions, the mathematical underpinnings of the ML models, and the architectural context required to reason about the system as a whole.

**Who should read these?**
- New engineers joining the project who need to understand the "big picture" before writing code.
- Product managers or stakeholders evaluating the technical approach.
- Hackathon judges evaluating the depth of the ML implementation.
- Researchers studying the system as a case study in government AI applications.

**Documents in this folder:**

| Document | Purpose |
|---|---|
| `problem-statement.md` | A macro-to-micro analysis of the DBT leakage problem in India. Covers the four core leakage typologies, regulatory impact, and why AI is the correct tool. |
| `system-overview-and-architecture.md` | Describes the decoupled microservices architecture: Next.js frontend, Python Flask ML Engine, PostgreSQL, and Redis. Includes data flow diagrams. |
| `ml-models-and-features.md` | In-depth documentation of all five ML models (Isolation Forest, Fuzzy Matching, Deterministic Heuristics, Prescriptive AI, Supervised Classification). Covers feature engineering, hyperparameter selection, and evaluation metrics. |
| `frontend-architecture.md` | Complete reference for the Next.js 14 frontend. Covers all 13 routes across three role-based route groups, the Framer Motion animation patterns, the TanStack Query state management strategy, and a detailed breakdown of the six security checks performed by `proxy.ts`. |
| `model-workflows.md` | Traces the exact journey of a transaction from CSV upload to AI-generated prescriptive suggestion. Includes step-by-step flow for each detection scenario. |
| `use-cases-and-workflows.md` | Role-centric workflows for the District Finance Officer, Field Verifier, and State Administrator. |
| `future-roadmap.md` | Planned Phase 2 and Phase 3 features including real-time streaming, blockchain audit trails, and mobile app development for field verifiers. |

---

### 3.2 `reference/`

The `reference/` folder contains **technical specifications**. These documents are intended for lookup, not linear reading. An engineer implementing a new API endpoint should not need to read the entire document — they should be able to jump directly to the specific contract they need.

**Who should read these?**
- Backend engineers implementing API endpoints against the specification.
- Data engineers designing ingestion pipelines against the schema.
- Frontend engineers validating API response shapes.
- Integration partners connecting external government data sources.

**Documents in this folder:**

| Document | Purpose |
|---|---|
| `data-schema-and-er.md` | The complete PostgreSQL schema including all table definitions, column types, constraints, foreign key relationships, and an Entity-Relationship (ER) diagram. |
| `leakage-rules-engine.md` | The formal, mathematical specification of every detection rule. Defines the exact threshold values, the additive Risk Score formula, and the dynamic Isolation Forest scoring equation. |
| `api-contracts.md` | Endpoint-by-endpoint documentation for all REST API routes. Includes method, path, request body schema, response shape, error codes, and authentication requirements. |

---

### 3.3 `tutorials/`

The `tutorials/` folder contains **learning-oriented** step-by-step guides designed for someone setting up the project for the first time. Unlike How-to Guides, tutorials assume no prior knowledge of this specific system and hold the reader's hand through each step.

**Who should read these?**
- New contributors cloning the repository for the first time.
- Hackathon judges who want to run the project locally to evaluate it.
- Government IT staff evaluating the system for a pilot deployment.

**Documents in this folder:**

| Document | Purpose |
|---|---|
| `getting-started.md` | A complete, beginner-friendly guide covering prerequisites, cloning the repo, installing Python ML dependencies (PyTorch, TensorFlow, scikit-learn), launching the Jupyter Notebooks, running the Flask ML Engine, and starting the Next.js frontend. |

---

### 3.4 `how-to/`

The `how-to/` folder contains **goal-oriented** guides for specific tasks. Unlike tutorials, how-to guides assume the reader already has the system running and needs to accomplish a particular objective.

**Who should read these?**
- DevOps engineers responsible for cloud deployment and scaling.
- System administrators configuring the application for a new district.

**Documents in this folder:**

| Document | Purpose |
|---|---|
| `deploy-and-scale.md` | Instructions for deploying the system to a cloud environment (AWS/GCP), configuring environment variables, setting up the PostgreSQL production database, and scaling the ML Engine horizontally behind a load balancer. |

---

## 4. Document Index

A flat index of every document for quick access:

| Document | Folder | Key Topics |
|---|---|---|
| [Problem Statement](explanation/problem-statement.md) | explanation | DBT leakage typologies, ghost beneficiaries, India policy context |
| [System Overview](explanation/system-overview-and-architecture.md) | explanation | Microservices, Next.js, Flask, PostgreSQL, Redis |
| [ML Models and Features](explanation/ml-models-and-features.md) | explanation | Isolation Forest, Fuzzy Matching, PyTorch, TensorFlow, Risk Scores |
| [Frontend Architecture](explanation/frontend-architecture.md) | explanation | Next.js routes, proxy.ts, Framer Motion, TanStack Query |
| [Model Workflows](explanation/model-workflows.md) | explanation | Data flow, transaction lifecycle, anomaly escalation |
| [Use Cases and Workflows](explanation/use-cases-and-workflows.md) | explanation | DFO workflow, Verifier workflow, Admin workflow |
| [Future Roadmap](explanation/future-roadmap.md) | explanation | Phase 2, Phase 3, blockchain audit, mobile app |
| [Data Schema and ER](reference/data-schema-and-er.md) | reference | PostgreSQL tables, ER diagram, column types |
| [Leakage Rules Engine](reference/leakage-rules-engine.md) | reference | Detection rules, Risk Score formula, thresholds |
| [API Contracts](reference/api-contracts.md) | reference | REST endpoints, request/response schemas, auth |
| [Getting Started](tutorials/getting-started.md) | tutorials | Setup, installation, running locally |
| [Deploy and Scale](how-to/deploy-and-scale.md) | how-to | Cloud deployment, env config, horizontal scaling |

---

## 5. Documentation Standards

All documents in this directory follow these conventions:

- **File Naming:** All files use kebab-case (e.g., `ml-models-and-features.md`).
- **Markdown Flavor:** GitHub Flavored Markdown (GFM). All tables, code blocks, and callouts must render correctly on GitHub.
- **Callout Blocks:** GitHub alert syntax is used for important information:
  - `> [!NOTE]` for background context.
  - `> [!IMPORTANT]` for critical requirements.
  - `> [!WARNING]` for breaking changes or risk.
- **Code Blocks:** All code examples must specify a language identifier (e.g., ` ```python `, ` ```bash `).
- **Table of Contents:** Every document longer than three sections must include a Table of Contents with anchor links.
