# API Documentation
## Unified Asset & Operations Brain

**Base URL:** `https://api.assetbrain.io/v1`
**Auth:** OAuth2 / OIDC Bearer Token (`Authorization: Bearer <token>`)
**Format:** JSON (REST); GraphQL endpoint available at `/graphql` for complex graph queries

---

## 1. Authentication

### `POST /auth/login`
Authenticates via SSO/OIDC redirect flow; returns session token.

### `POST /auth/refresh`
Refreshes an expiring access token.

---

## 2. Document Ingestion APIs

### `POST /documents`
Upload a document for ingestion.
**Request:** `multipart/form-data`
```json
{
  "file": "<binary>",
  "plant_id": "uuid",
  "doc_type_hint": "work_order | drawing | SOP | inspection | regulatory | other (optional)"
}
```
**Response `202 Accepted`:**
```json
{
  "document_id": "uuid",
  "ingestion_status": "pending",
  "job_id": "uuid"
}
```

### `GET /documents/{document_id}`
Returns document metadata, ingestion status, extracted entity summary.

### `GET /documents`
Query params: `plant_id, doc_type, status, page, page_size`
Returns paginated document list.

### `GET /documents/{document_id}/entities`
Returns extracted entities for a document.

### `GET /ingestion-jobs/{job_id}`
Returns ingestion job status and any error details.

---

## 3. Knowledge Copilot APIs

### `POST /copilot/query`
Submit a natural-language query.
**Request:**
```json
{
  "query": "What is the last inspection finding for pump P-204?",
  "conversation_id": "uuid (optional)",
  "plant_id": "uuid"
}
```
**Response `200 OK`:**
```json
{
  "answer": "The last inspection on P-204 (12-May-2026) found minor seal wear...",
  "confidence": 0.91,
  "citations": [
    {"document_id": "uuid", "title": "Inspection Report P-204 May 2026", "chunk_reference": "p.3"}
  ],
  "requires_review": false,
  "conversation_id": "uuid"
}
```

### `GET /copilot/conversations/{conversation_id}`
Returns full conversation history with citations.

### `POST /copilot/feedback`
Submit thumbs up/down + comments on an answer.
```json
{ "message_id": "uuid", "helpful": true, "comment": "optional" }
```

---

## 4. Maintenance & RCA APIs

### `GET /equipment/{equipment_id}/history`
Returns work orders, inspections, and failure records for equipment.

### `POST /rca/generate`
Auto-generate an RCA draft from historical data.
```json
{ "equipment_id": "uuid", "failure_id": "uuid (optional)", "problem_statement": "string" }
```
**Response:**
```json
{
  "rca_id": "uuid",
  "suggested_contributing_factors": ["..."],
  "related_failures": ["failure_id", "..."],
  "confidence": 0.85
}
```

### `GET /maintenance/predictions?equipment_id={id}`
Returns predictive maintenance recommendations with justification and confidence.

### `GET /maintenance/schedule/optimized?plant_id={id}`
Returns AI-optimized maintenance schedule.

---

## 5. Compliance APIs

### `GET /compliance/gaps`
Query params: `plant_id, severity, regulation_source, status`
Returns list of detected compliance gaps.

### `GET /compliance/gaps/{gap_id}`
Returns gap detail including mapped regulation clause and affected equipment/procedure.

### `POST /compliance/evidence-package`
Generate an audit evidence package.
```json
{ "gap_id": "uuid (optional)", "regulation_source": "OISD (optional)", "plant_id": "uuid" }
```
**Response:**
```json
{ "package_id": "uuid", "file_url": "https://.../evidence_package.pdf", "generated_at": "timestamp" }
```

---

## 6. Lessons Learned APIs

### `GET /incidents`
Query params: `plant_id, type, date_from, date_to`

### `POST /incidents`
Report a new incident/near-miss.

### `GET /incidents/patterns`
Returns AI-detected systemic patterns across incidents.

### `GET /alerts`
Returns proactive warnings pushed to the current user's role/plant.

---

## 7. Knowledge Graph API (GraphQL)

**Endpoint:** `POST /graphql`

Example query — equipment relationship traversal:
```graphql
query {
  equipment(tagNumber: "P-204") {
    description
    hasProcedure { title }
    involvedIn { incidentId, type, date }
    partOf { tagNumber }
  }
}
```

---

## 8. Admin APIs

### `GET /admin/users` / `POST /admin/users` / `PATCH /admin/users/{id}`
Standard user management with role assignment.

### `POST /admin/connectors`
Configure an ingestion connector (email, CMMS, ERP).

### `GET /admin/audit-logs`
Query params: `user_id, action, date_from, date_to`

---

## 9. Common Response Envelope & Error Handling

**Standard error response:**
```json
{
  "error": {
    "code": "DOCUMENT_NOT_FOUND",
    "message": "No document found with the given ID",
    "request_id": "uuid"
  }
}
```

| HTTP Status | Meaning |
|---|---|
| 200 | Success |
| 202 | Accepted (async processing) |
| 400 | Bad Request / validation error |
| 401 | Unauthorized |
| 403 | Forbidden (RBAC) |
| 404 | Not Found |
| 429 | Rate Limited |
| 500 | Internal Server Error |

---

## 10. Rate Limiting
- Default: 100 requests/minute per user token
- Copilot query endpoint: 20 requests/minute per user (LLM cost control)
- Bulk ingestion endpoint: 10 concurrent uploads per tenant
