# Project Structure Document
## Unified Asset & Operations Brain

---

## 1. Repository Layout (Monorepo)

```
asset-operations-brain/
├── README.md
├── docs/
│   ├── 01_HLD.md
│   ├── 02_SRS.md
│   ├── 03_Project_Plan_Roadmap.md
│   ├── 04_UIUX_Design.md
│   ├── 05_System_Architecture.md
│   ├── 06_Database_Schema_ERD.md
│   ├── 07_API_Doc.md
│   └── 08_Project_Structure.md
│
├── infra/
│   ├── terraform/
│   │   ├── modules/ (vpc, eks, rds, kafka, neo4j, qdrant)
│   │   └── environments/ (dev, staging, prod)
│   ├── k8s/
│   │   ├── base/
│   │   └── overlays/ (dev, staging, prod)
│   └── docker-compose.yml         # local dev stack
│
├── services/
│   ├── ingestion-service/
│   │   ├── src/
│   │   │   ├── connectors/ (file, email, cmms, erp)
│   │   │   ├── ocr/
│   │   │   ├── classifiers/
│   │   │   └── main.py
│   │   ├── tests/
│   │   ├── Dockerfile
│   │   └── requirements.txt
│   │
│   ├── entity-extraction-service/
│   │   ├── src/
│   │   │   ├── ner_models/
│   │   │   ├── prompts/
│   │   │   └── main.py
│   │   ├── tests/
│   │   └── Dockerfile
│   │
│   ├── knowledge-graph-service/
│   │   ├── src/
│   │   │   ├── graph_builder/
│   │   │   ├── ontology/
│   │   │   └── main.py
│   │   ├── tests/
│   │   └── Dockerfile
│   │
│   ├── embedding-service/
│   │   ├── src/
│   │   ├── tests/
│   │   └── Dockerfile
│   │
│   ├── copilot-agent-service/
│   │   ├── src/
│   │   │   ├── retrieval/ (vector + graph hybrid)
│   │   │   ├── reranker/
│   │   │   ├── prompts/
│   │   │   ├── confidence_scoring/
│   │   │   └── main.py
│   │   ├── tests/
│   │   └── Dockerfile
│   │
│   ├── maintenance-rca-agent-service/
│   │   ├── src/
│   │   ├── tests/
│   │   └── Dockerfile
│   │
│   ├── compliance-agent-service/
│   │   ├── src/
│   │   │   ├── regulation_mapper/
│   │   │   ├── gap_detector/
│   │   │   └── evidence_generator/
│   │   ├── tests/
│   │   └── Dockerfile
│   │
│   ├── lessons-learned-agent-service/
│   │   ├── src/
│   │   │   ├── pattern_mining/
│   │   │   └── alerting/
│   │   ├── tests/
│   │   └── Dockerfile
│   │
│   ├── api-gateway/
│   │   ├── src/
│   │   ├── tests/
│   │   └── Dockerfile
│   │
│   └── auth-service/
│       ├── src/
│       ├── tests/
│       └── Dockerfile
│
├── frontend/
│   ├── web-app/
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── pages/ (Copilot, Maintenance, Compliance, LessonsLearned, Admin)
│   │   │   ├── hooks/
│   │   │   ├── services/ (API clients)
│   │   │   ├── store/
│   │   │   └── App.tsx
│   │   ├── public/
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── mobile-pwa/
│       ├── src/
│       ├── public/
│       └── package.json
│
├── shared/
│   ├── proto/ or schemas/          # shared API/event schemas
│   ├── ontology/                   # industrial ontology definitions (ISO 15926 aligned)
│   └── utils/
│
├── data/
│   ├── sample-corpus/              # sample P&IDs, work orders, SOPs for demo
│   ├── benchmark-qna/              # domain-expert benchmark questions for eval
│   └── seed-scripts/
│
├── tests/
│   ├── integration/
│   └── e2e/
│
├── .github/
│   └── workflows/ (ci.yml, cd.yml, lint.yml)
│
├── scripts/
│   ├── setup_local_env.sh
│   ├── seed_demo_data.py
│   └── run_benchmarks.py
│
└── .env.example
```

---

## 2. Module Ownership Mapping (aligned to Challenge Brief modules)

| Challenge Brief Module | Corresponding Service(s) |
|---|---|
| Universal Document Ingestion & Knowledge Graph Agent | `ingestion-service`, `entity-extraction-service`, `knowledge-graph-service` |
| Expert Knowledge Copilot | `copilot-agent-service`, `frontend/web-app`, `frontend/mobile-pwa` |
| Maintenance Intelligence & RCA Agent | `maintenance-rca-agent-service` |
| Quality & Regulatory Compliance Intelligence | `compliance-agent-service` |
| Lessons Learned & Failure Intelligence Engine | `lessons-learned-agent-service` |

---

## 3. Local Development Setup

```bash
# Clone and bootstrap
git clone <repo-url>
cd asset-operations-brain
cp .env.example .env

# Start local infra (Postgres, Neo4j, Qdrant, Kafka, Redis)
docker-compose -f infra/docker-compose.yml up -d

# Seed demo data
python scripts/seed_demo_data.py

# Run backend services (example: copilot agent)
cd services/copilot-agent-service
pip install -r requirements.txt --break-system-packages
python src/main.py

# Run frontend
cd frontend/web-app
npm install
npm run dev
```

---

## 4. Branching & CI/CD Strategy

- **Branching model:** trunk-based with short-lived feature branches (`feature/*`, `fix/*`)
- **CI:** GitHub Actions — lint, unit test, build, container scan on every PR
- **CD:** auto-deploy to `dev` on merge to `main`; manual promotion to `staging`/`prod`
- **Versioning:** Semantic versioning per service; API versioned via URL path (`/v1`)

---

## 5. Coding Standards

| Layer | Standard |
|---|---|
| Python services | PEP8, type hints, `black` + `ruff` formatting/linting |
| Frontend | ESLint + Prettier, strict TypeScript |
| API contracts | OpenAPI 3.1 spec per service, generated client SDKs |
| Commits | Conventional Commits (`feat:`, `fix:`, `docs:`, etc.) |
| Testing | pytest (backend), Jest/React Testing Library (frontend), min. 80% coverage on core logic |
