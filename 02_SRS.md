# Software Requirements Specification (SRS)
## Project: Unified Asset & Operations Brain

**Version:** 1.0 | **Standard Reference:** IEEE 830 / ISO/IEC/IEEE 29148

---

## 1. Introduction

### 1.1 Purpose
Defines functional and non-functional requirements for the Industrial Knowledge Intelligence
platform, targeted at plant engineers, maintenance technicians, quality/compliance officers, and
plant management in asset-intensive industries (energy, manufacturing, process plants).

### 1.2 Intended Audience
Engineering team, hackathon judges, product stakeholders, QA team.

### 1.3 Product Scope
See HLD Section 4 (In Scope / Out of Scope).

### 1.4 Definitions & Acronyms

| Term | Definition |
|---|---|
| P&ID | Piping & Instrumentation Diagram |
| RAG | Retrieval-Augmented Generation |
| RCA | Root Cause Analysis |
| OISD | Oil Industry Safety Directorate |
| PESO | Petroleum and Explosives Safety Organisation |
| QMS | Quality Management System |
| KG | Knowledge Graph |
| OCR | Optical Character Recognition |

---

## 2. Overall Description

### 2.1 Product Perspective
A greenfield, standalone platform that integrates with (but does not replace) existing CMMS, ERP,
and document management systems via connectors/APIs.

### 2.2 User Classes and Characteristics

| User Class | Description | Technical Proficiency |
|---|---|---|
| Field Technician | Uses mobile copilot for procedures, equipment history | Low–Medium |
| Maintenance Engineer | Uses RCA/predictive maintenance module | Medium |
| Quality/Compliance Officer | Uses compliance intelligence, audit packages | Medium–High |
| Plant Manager | Dashboards, KPIs, cross-functional insights | Medium |
| System Administrator | Manages ingestion, users, access control | High |

### 2.3 Operating Environment
- Web browsers (Chrome, Edge, Safari — latest 2 versions)
- Mobile: responsive PWA, Android/iOS via installable web app
- Cloud-hosted backend (Kubernetes cluster), on-prem edge cache optional for low-connectivity sites

### 2.4 Design & Implementation Constraints
- Must support offline/degraded-connectivity field usage (cached procedures, queued queries)
- Must maintain full audit trail per regulatory requirement
- LLM outputs for safety/compliance-critical content require human review flag before being marked "approved"

---

## 3. Functional Requirements

### FR-1 Document Ingestion
- FR-1.1: System shall ingest PDF, DOCX, XLSX, scanned images, email archives (.eml/.pst), and CAD/P&ID exports
- FR-1.2: System shall auto-classify document type (drawing, work order, procedure, inspection report, regulatory filing)
- FR-1.3: System shall run OCR on scanned/handwritten documents with confidence scoring
- FR-1.4: System shall detect and flag duplicate or superseded document versions

### FR-2 Entity Extraction & Knowledge Graph
- FR-2.1: System shall extract equipment tags, process parameters, personnel names, dates, regulatory references
- FR-2.2: System shall construct/update a knowledge graph linking entities across document types
- FR-2.3: System shall version the knowledge graph and support point-in-time queries
- FR-2.4: System shall automatically re-index and re-link the graph when new documents arrive (event-driven)

### FR-3 Expert Knowledge Copilot
- FR-3.1: System shall answer natural-language queries using RAG over the full document corpus
- FR-3.2: Every answer shall include source citations, a confidence score, and a deep link to the originating document
- FR-3.3: Copilot shall be usable on mobile devices with a simplified, low-bandwidth UI
- FR-3.4: System shall support voice input for hands-busy field scenarios
- FR-3.5: System shall escalate low-confidence answers to a "needs human review" queue

### FR-4 Maintenance Intelligence & RCA
- FR-4.1: System shall fuse work order history, failure records, OEM manuals, and inspection findings
- FR-4.2: System shall generate predictive maintenance recommendations with justification
- FR-4.3: System shall support guided RCA workflows (5-Why, Fishbone) pre-populated with relevant historical data
- FR-4.4: System shall propose optimized maintenance schedules based on failure pattern analysis

### FR-5 Quality & Regulatory Compliance
- FR-5.1: System shall map regulatory requirements (Factory Act, OISD, PESO, environmental norms) to current procedures/equipment state
- FR-5.2: System shall identify and rank compliance gaps by risk severity
- FR-5.3: System shall auto-generate audit evidence packages (PDF/DOCX export)
- FR-5.4: System shall flag quality deviations proactively based on incoming inspection data

### FR-6 Lessons Learned & Failure Intelligence
- FR-6.1: System shall mine incident reports, near-misses, audit findings, and non-conformances for systemic patterns
- FR-6.2: System shall cross-reference external industry incident databases (where accessible)
- FR-6.3: System shall proactively push relevant warnings to operational teams when similar conditions recur

### FR-7 Administration & Governance
- FR-7.1: System shall support role-based access control (RBAC)
- FR-7.2: System shall log all queries, answers, and document access for audit purposes
- FR-7.3: System shall allow administrators to configure ingestion connectors and retention policies

---

## 4. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Performance | 95th-percentile query response < 5s for cached/indexed content |
| Scalability | Support ≥ 500 concurrent users and ≥ 1M documents per tenant |
| Availability | 99.5% uptime SLA |
| Security | AES-256 at rest, TLS 1.3 in transit, OIDC/OAuth2 auth, RBAC |
| Compliance | Data residency configurable (India region for regulated data) |
| Usability | Mobile-first copilot; WCAG 2.1 AA accessibility target |
| Maintainability | Modular microservice architecture, >80% automated test coverage on core services |
| Auditability | Immutable audit log of AI answers vs. source documents |

## 5. External Interface Requirements
- REST/GraphQL APIs (see API Doc)
- CMMS/ERP connector interfaces (SAP PM, Maximo — read initially, write in Phase 2)
- SSO integration (Azure AD / Okta)
- Email ingestion via IMAP/Graph API

## 6. Acceptance Criteria (Hackathon-Aligned)
Mapped directly to Evaluation Focus in the challenge brief:
1. Entity extraction accuracy benchmark passed on sample document set
2. Query answer quality validated against domain-expert Q&A set
3. Knowledge graph linkage completeness verified
4. Time-to-answer measurably faster than manual search baseline
5. Compliance gap detection validated against known-gap test cases
6. Demonstrated improvement in cross-functional knowledge discovery (case study/demo)
