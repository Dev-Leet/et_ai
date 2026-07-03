# Database Schema & ERD Document
## Unified Asset & Operations Brain

The platform uses a **polyglot persistence** model:
- **PostgreSQL** — transactional/relational metadata (users, documents, workflows, audit)
- **Neo4j** — knowledge graph (equipment, entities, relationships)
- **Qdrant / pgvector** — vector embeddings for semantic search

---

## 1. Relational Schema (PostgreSQL)

### 1.1 Entity Relationship Diagram (textual)

```
Users ──< UserRoles >── Roles
  │
  └──< AuditLogs

Documents ──< DocumentVersions
Documents ──< ExtractedEntities
Documents ──< IngestionJobs
Documents }──{ Equipment (via DocumentEquipmentLink)

Equipment ──< WorkOrders
Equipment ──< InspectionRecords
Equipment ──< FailureRecords

WorkOrders ──< RCARecords

ComplianceRequirements ──< ComplianceGaps }──{ Equipment
ComplianceGaps ──< AuditEvidencePackages

Incidents ──< IncidentPatterns
Incidents }──{ Equipment

Conversations ──< Messages
Messages ──< MessageCitations }──{ Documents
```

### 1.2 Core Tables

#### `users`
| Column | Type | Notes |
|---|---|---|
| user_id | UUID (PK) | |
| name | VARCHAR | |
| email | VARCHAR (UNIQUE) | |
| role_id | UUID (FK → roles) | |
| plant_id | UUID (FK → plants) | |
| created_at | TIMESTAMP | |

#### `roles`
| Column | Type | Notes |
|---|---|---|
| role_id | UUID (PK) | |
| role_name | VARCHAR | e.g., Technician, Engineer, Compliance Officer, Admin |
| permissions | JSONB | RBAC permission set |

#### `plants`
| Column | Type | Notes |
|---|---|---|
| plant_id | UUID (PK) | |
| name | VARCHAR | |
| location | VARCHAR | |
| region | VARCHAR | for data residency |

#### `documents`
| Column | Type | Notes |
|---|---|---|
| document_id | UUID (PK) | |
| title | VARCHAR | |
| doc_type | VARCHAR | drawing / work_order / SOP / inspection / regulatory / other |
| source_system | VARCHAR | upload / email / CMMS / ERP |
| storage_path | VARCHAR | object storage URI |
| plant_id | UUID (FK → plants) | |
| uploaded_by | UUID (FK → users) | |
| ingestion_status | VARCHAR | pending / processing / completed / failed |
| ocr_confidence | FLOAT | |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |

#### `document_versions`
| Column | Type | Notes |
|---|---|---|
| version_id | UUID (PK) | |
| document_id | UUID (FK → documents) | |
| version_number | INT | |
| storage_path | VARCHAR | |
| is_superseded | BOOLEAN | |
| created_at | TIMESTAMP | |

#### `extracted_entities`
| Column | Type | Notes |
|---|---|---|
| entity_id | UUID (PK) | |
| document_id | UUID (FK → documents) | |
| entity_type | VARCHAR | equipment_tag / parameter / person / date / regulation_ref |
| entity_value | VARCHAR | |
| confidence | FLOAT | |
| kg_node_ref | VARCHAR | reference ID in Neo4j |

#### `equipment`
| Column | Type | Notes |
|---|---|---|
| equipment_id | UUID (PK) | |
| tag_number | VARCHAR (UNIQUE) | |
| description | VARCHAR | |
| equipment_type | VARCHAR | |
| plant_id | UUID (FK → plants) | |
| parent_equipment_id | UUID (FK → equipment, nullable) | for hierarchy |
| commissioned_date | DATE | |

#### `work_orders`
| Column | Type | Notes |
|---|---|---|
| work_order_id | UUID (PK) | |
| equipment_id | UUID (FK → equipment) | |
| type | VARCHAR | preventive / corrective / predictive |
| status | VARCHAR | open / closed / in_progress |
| description | TEXT | |
| created_at | TIMESTAMP | |
| closed_at | TIMESTAMP | |

#### `inspection_records`
| Column | Type | Notes |
|---|---|---|
| inspection_id | UUID (PK) | |
| equipment_id | UUID (FK → equipment) | |
| inspector_id | UUID (FK → users) | |
| findings | TEXT | |
| severity | VARCHAR | |
| inspection_date | DATE | |
| document_id | UUID (FK → documents, nullable) | |

#### `failure_records`
| Column | Type | Notes |
|---|---|---|
| failure_id | UUID (PK) | |
| equipment_id | UUID (FK → equipment) | |
| failure_mode | VARCHAR | |
| root_cause | TEXT | |
| downtime_hours | FLOAT | |
| occurred_at | TIMESTAMP | |

#### `rca_records`
| Column | Type | Notes |
|---|---|---|
| rca_id | UUID (PK) | |
| work_order_id | UUID (FK → work_orders, nullable) | |
| failure_id | UUID (FK → failure_records, nullable) | |
| problem_statement | TEXT | |
| contributing_factors | JSONB | |
| root_cause | TEXT | |
| corrective_action | TEXT | |
| created_by | UUID (FK → users) | |
| created_at | TIMESTAMP | |

#### `compliance_requirements`
| Column | Type | Notes |
|---|---|---|
| requirement_id | UUID (PK) | |
| regulation_source | VARCHAR | Factory Act / OISD / PESO / Environmental |
| clause_reference | VARCHAR | |
| description | TEXT | |

#### `compliance_gaps`
| Column | Type | Notes |
|---|---|---|
| gap_id | UUID (PK) | |
| requirement_id | UUID (FK → compliance_requirements) | |
| equipment_id | UUID (FK → equipment, nullable) | |
| severity | VARCHAR | low / medium / high / critical |
| description | TEXT | |
| status | VARCHAR | open / in_progress / resolved |
| detected_at | TIMESTAMP | |

#### `audit_evidence_packages`
| Column | Type | Notes |
|---|---|---|
| package_id | UUID (PK) | |
| gap_id | UUID (FK → compliance_gaps, nullable) | |
| generated_by | UUID (FK → users) | |
| file_path | VARCHAR | |
| generated_at | TIMESTAMP | |

#### `incidents`
| Column | Type | Notes |
|---|---|---|
| incident_id | UUID (PK) | |
| type | VARCHAR | incident / near_miss / non_conformance |
| description | TEXT | |
| equipment_id | UUID (FK → equipment, nullable) | |
| plant_id | UUID (FK → plants) | |
| occurred_at | TIMESTAMP | |
| document_id | UUID (FK → documents, nullable) | |

#### `incident_patterns`
| Column | Type | Notes |
|---|---|---|
| pattern_id | UUID (PK) | |
| description | TEXT | |
| related_incident_ids | UUID[] | |
| confidence | FLOAT | |
| detected_at | TIMESTAMP | |

#### `conversations` / `messages` / `message_citations`
| Table | Key Columns |
|---|---|
| conversations | conversation_id (PK), user_id (FK), started_at |
| messages | message_id (PK), conversation_id (FK), role (user/assistant), content, confidence_score, created_at |
| message_citations | citation_id (PK), message_id (FK), document_id (FK), chunk_reference |

#### `audit_logs`
| Column | Type | Notes |
|---|---|---|
| log_id | UUID (PK) | |
| user_id | UUID (FK → users) | |
| action | VARCHAR | query / document_access / override / export |
| target_reference | VARCHAR | |
| timestamp | TIMESTAMP | |

---

## 2. Knowledge Graph Schema (Neo4j)

### Node Labels
- `Equipment {tag_number, type, description}`
- `Document {document_id, doc_type, title}`
- `Person {name, role}`
- `Regulation {source, clause_reference}`
- `Incident {incident_id, type, date}`
- `Procedure {procedure_id, title}`
- `Parameter {name, unit}`

### Relationship Types
- `(Equipment)-[:PART_OF]->(Equipment)` — hierarchy
- `(Document)-[:MENTIONS]->(Equipment)`
- `(Document)-[:MENTIONS]->(Person)`
- `(Document)-[:REFERENCES]->(Regulation)`
- `(Equipment)-[:HAS_PROCEDURE]->(Procedure)`
- `(Equipment)-[:INVOLVED_IN]->(Incident)`
- `(Equipment)-[:HAS_PARAMETER]->(Parameter)`
- `(Regulation)-[:APPLIES_TO]->(Equipment)`
- `(Incident)-[:SIMILAR_TO]->(Incident)` — derived via pattern mining

---

## 3. Vector Store Schema (Qdrant)

### Collection: `document_chunks`
| Field | Type | Notes |
|---|---|---|
| id | UUID | chunk identifier |
| vector | float[1536] | embedding |
| document_id | UUID | payload, links to PostgreSQL `documents` |
| chunk_text | string | payload |
| chunk_index | int | payload |
| doc_type | string | payload, used for filtered search |
| plant_id | UUID | payload, tenant isolation filter |

---

## 4. Indexing Strategy

- PostgreSQL: B-tree indexes on all FKs, GIN index on `extracted_entities.entity_value` for text search
- Neo4j: indexes on `Equipment.tag_number`, `Document.document_id`, `Regulation.clause_reference`
- Qdrant: HNSW index on vector field, payload index on `plant_id` and `doc_type` for filtered hybrid search
