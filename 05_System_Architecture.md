# System Architecture Document
## Unified Asset & Operations Brain

---

## 1. Architecture Style

Microservices-based, event-driven architecture with a hybrid retrieval layer (vector + graph),
orchestrated agentic AI components, and API-first integration surface.

---

## 2. High-Level Architecture Diagram (textual)

```
                                   ┌───────────────────────────┐
                                   │        Client Layer        │
                                   │  Web App (React) | PWA     │
                                   │  Mobile (installable PWA)  │
                                   └─────────────┬───────────────┘
                                                 │ REST/GraphQL (HTTPS, OAuth2)
                                   ┌─────────────▼───────────────┐
                                   │        API Gateway           │
                                   │  (Auth, Rate Limit, Routing) │
                                   └─────────────┬───────────────┘
                     ┌──────────────────┬────────┼────────┬──────────────────┐
                     ▼                  ▼        ▼        ▼                  ▼
             ┌──────────────┐  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
             │ Copilot Agent│  │ Maintenance/ │ │ Compliance   │ │ Lessons      │
             │ Service (RAG)│  │ RCA Agent    │ │ Agent        │ │ Learned Agent│
             └──────┬───────┘  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
                     └──────────────────┴────────┬───────┴──────────────────┘
                                                 ▼
                                   ┌───────────────────────────┐
                                   │   Agent Orchestration       │
                                   │   (LangGraph / MCP Tools)   │
                                   └─────────────┬───────────────┘
                                                 ▼
                     ┌──────────────────────────┼──────────────────────────┐
                     ▼                          ▼                          ▼
             ┌──────────────┐          ┌──────────────┐           ┌──────────────┐
             │ Vector Store │          │ Knowledge     │           │ Relational DB │
             │ (Qdrant)     │          │ Graph (Neo4j) │           │ (PostgreSQL)  │
             └──────┬───────┘          └──────┬───────┘           └──────┬───────┘
                     └──────────────────────────┴──────────────────────────┘
                                                 ▲
                                   ┌─────────────┴───────────────┐
                                   │   Knowledge Processing Layer │
                                   │  OCR | Entity Extraction |   │
                                   │  Embedding Gen | KG Builder  │
                                   └─────────────┬───────────────┘
                                                 ▲
                                   ┌─────────────┴───────────────┐
                                   │      Ingestion Layer          │
                                   │ Connectors: File Upload, Email│
                                   │ (IMAP/Graph), CMMS/ERP, Scans │
                                   └─────────────┬───────────────┘
                                                 ▲
                                   ┌─────────────┴───────────────┐
                                   │  Source Systems: CMMS, ERP,  │
                                   │  Email, Shared Drives, Scans │
                                   └───────────────────────────────┘

        Cross-cutting: Message Bus (Kafka) | Object Storage (S3/Blob) |
        IAM/RBAC | Observability (OpenTelemetry+Grafana) | Audit Log Service
```

---

## 3. Component Descriptions

### 3.1 Ingestion Layer
- **File Connector Service:** handles upload, watch-folder, and batch import (PDF, DOCX, XLSX, images)
- **Email Connector:** IMAP/Microsoft Graph API polling for attachments
- **CMMS/ERP Connector:** REST/OData pull from SAP PM / Maximo (read-only Phase 1)
- Publishes ingestion events to Kafka topic `doc.ingested`

### 3.2 Knowledge Processing Layer
- **OCR/Document AI Service:** text + layout extraction; P&ID parser (CV model for symbol/tag detection)
- **Classification Service:** document-type classifier (drawing, work order, SOP, inspection, regulatory)
- **Entity Extraction Service:** NER model (fine-tuned/prompted LLM) extracting equipment tags, parameters, personnel, dates, regulatory refs
- **Embedding Service:** generates vector embeddings for chunked document content
- **Knowledge Graph Builder:** resolves entities, creates/updates nodes & relationships in Neo4j, handles versioning

### 3.3 Knowledge Layer
- **Vector Store (Qdrant/pgvector):** semantic search over document chunks
- **Knowledge Graph (Neo4j):** entity relationships, equipment hierarchies, regulatory mappings
- **Relational DB (PostgreSQL):** metadata, users, RBAC, audit logs, workflow state

### 3.4 Agentic Layer
- **Copilot Agent:** RAG pipeline — retrieve (vector+graph hybrid) → rerank → generate with citations → confidence scoring
- **Maintenance/RCA Agent:** combines structured work-order data + retrieved unstructured context → predictive recommendations
- **Compliance Agent:** maps regulatory corpus to current procedures/equipment; gap scoring
- **Lessons Learned Agent:** clustering/pattern-mining over incident corpus; proactive alert generation
- **Orchestration:** LangGraph-based multi-agent coordination; tool calls exposed via MCP for enterprise system actions

### 3.5 Application Layer
- API Gateway (auth, rate limiting, request routing)
- Web App (React/TypeScript)
- Mobile PWA (offline cache, voice input)

### 3.6 Cross-Cutting Concerns
- **Message Bus (Kafka):** event-driven pipeline decoupling
- **Object Storage (S3/Azure Blob):** raw document storage
- **IAM/RBAC:** OIDC-based auth, role-scoped data access
- **Observability:** distributed tracing, metrics, structured logs
- **Audit Log Service:** immutable record of queries, answers, source links, and human overrides

---

## 4. Data Flow (Query Path)

1. User submits query via Copilot (web/mobile)
2. API Gateway authenticates and routes to Copilot Agent Service
3. Agent performs hybrid retrieval: vector similarity search + graph traversal for related entities
4. Retrieved chunks reranked and passed to LLM with strict citation-grounding prompt
5. LLM generates answer + confidence score; response includes source document links
6. Answer + query logged to Audit Log Service
7. Low-confidence answers flagged and routed to human-review queue

## 5. Data Flow (Ingestion Path)

1. Document arrives via connector → stored in Object Storage → event published to Kafka
2. OCR/Document AI extracts text/layout → Classification Service tags document type
3. Entity Extraction Service pulls structured entities
4. Embedding Service chunks + embeds content → stored in Vector Store
5. Knowledge Graph Builder resolves/links entities → updates Neo4j
6. Ingestion status updated in PostgreSQL; UI reflects new document availability

---

## 6. Deployment Architecture

- Containerized microservices (Docker) orchestrated via Kubernetes (AKS/EKS)
- Environments: Dev → Staging → Production, promoted via GitHub Actions CI/CD
- Autoscaling for ingestion workers and Copilot agent pods based on queue depth/load
- Regional data residency configuration for regulated industrial data (India region)
- Edge caching option for low-connectivity plant sites (local read replica of frequently accessed procedures)

## 7. Security Architecture

- OIDC/OAuth2 authentication, JWT-based session tokens
- RBAC enforced at API Gateway and service layer
- Encryption: TLS 1.3 in transit, AES-256 at rest
- Secrets managed via vault (HashiCorp Vault / cloud KMS)
- Audit trail immutable and exportable for regulatory review

## 8. Scalability Considerations

- Stateless agent services scale horizontally behind the API Gateway
- Vector store and graph DB sharded/clustered for multi-plant, multi-tenant scale
- Kafka partitioning by document source/tenant for parallel ingestion throughput
- Caching layer (Redis) for frequent Copilot queries to reduce LLM call volume/cost
