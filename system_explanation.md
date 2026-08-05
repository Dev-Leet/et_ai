# Industrial Knowledge Intelligence — System Walkthrough

This document explains, step-by-step, how the platform behaves in practice — starting from the moment a document enters the system, through processing and storage, to the moment a person receives an answer (or a warning they never had to ask for). It maps directly onto the two diagrams: **`architecture_animated.html`** (the five-layer structural view) and its embedded **workflow diagram** (the granular, phase-by-phase trace).

There are actually **two entry points** into the system, and the walkthrough below covers both:

1. **The document path** — something is uploaded, and the system has to understand it (Phases 1–4).
2. **The question path** — a person asks something, and the system has to answer it (Phase 5).

These two paths share the same underlying knowledge base, which is the whole point of the architecture: nothing a technician asks is ever answered from a single document in isolation — it's answered from everything the system has learned so far.

---

## PATH A — From Uploaded Document to Stored Knowledge

### Phase 1 — Capture & Ingestion *(Architecture Layer 1)*

**Trigger:** An engineer, inspector, or automated feed uploads a file — an inspection report, work order, scanned form, or engineering drawing.

**Step 1a — Upload.**
The file lands in **Supabase Storage**. This is intentionally the *only* front door for raw files — whether it came from a person dragging a PDF into the app, or an automated export from another system, it all converges here. The upload returns a stored object URL.

**Step 1b — Event Trigger.**
The moment that file is written to storage, a **Supabase Edge Function** fires automatically. Nothing is polling a folder; nothing waits for a scheduled job to notice the file exists. This is what makes the pipeline feel "alive" rather than batch-processed — ingestion and processing are decoupled by an event, not a clock.

**Step 1c — Type Classification.**
Before any heavy processing happens, a fast pass through **Gemini 1.5 Flash** looks at the document and tags what *kind* of thing it is — a P&ID, an inspection report, a work order, an email thread. This classification is what determines which processing path the document takes next. A drawing and a scanned inspection form need very different treatment, and this step is the fork in the road.

*→ Output of Phase 1: a stored file, a queued processing job, and a document type label.*

---

### Phase 2 — Extraction & Understanding *(Architecture Layer 2)*

This is where unstructured content — pixels, scanned handwriting, freeform text — becomes structured, queryable facts. The architecture splits this into two internal lanes: **"Read the Page"** and **"Understand It."**

**Step 2a — OCR / Layout Parsing.**
Depending on the document type from Phase 1: text-heavy PDFs go to **LlamaParse**, which preserves layout — tables stay tables, headers stay headers. Scanned or handwritten pages go to **Google Cloud Vision**, which is stronger on messy, non-digital-native input. Either way, the output is plain text plus preserved structure.

**Step 2b — Drawing / Symbol Detection.**
If the document is a P&ID or engineering drawing, it takes a separate branch: **OpenCV** first cleans up lines and contours, then **YOLOv8** detects and classifies symbols — valves, pumps, instruments — and records their coordinates on the page. This step doesn't apply to text documents; it's specific to schematic content.

**Step 2c — Entity Extraction.**
Whatever text came out of 2a (or 2b's tag-reading pass) gets sent to **Gemini 1.5 Flash in JSON mode** with a structured schema prompt. This is the step that turns "a paragraph describing what an inspector found" into typed fields: equipment tag, inspector name, date, finding description, severity. This structured JSON is what everything downstream depends on — it's the handoff point from "text" to "facts."

**Step 2d — Chunking.**
In parallel, the *full* text (not just the extracted entities) is split into semantically coherent chunks using **LangChain / LlamaIndex** splitters, sized and overlapped for good retrieval later. This preserves the narrative detail that structured extraction alone would lose — the entities capture *what* happened, the chunks preserve *how it was described*.

*→ Output of Phase 2: structured entity JSON + retrieval-ready text chunks, running in parallel.*

---

### Phase 3 — Knowledge Representation *(Architecture Layer 3)*

This is the layer that gives the platform its actual intelligence advantage — it's the difference between "a folder of processed PDFs" and a system that can connect facts across documents. Three internal lanes: **Relationships**, **Meaning**, and **Keep in Sync**.

**Step 3a — Graph Write.**
The entity JSON from Phase 2 becomes nodes and edges in **Neo4j AuraDB**. An inspection finding doesn't just get stored — it gets *connected*: `Equipment → HAS_FINDING → Inspection → PERFORMED_BY → Inspector`. This relationship structure is what later lets the RCA agent trace a failure back through every prior touchpoint with that piece of equipment, something a flat document search could never do.

**Step 3b — Embedding.**
Separately, the text chunks from Phase 2 are converted into dense vector embeddings via the **Gemini Embedding API** — a numerical representation of *meaning*, not just keywords, so a search for "leak" can also surface a document that said "seepage" or "loss of containment."

**Step 3c — Vector Write.**
Those embeddings are stored in **Supabase pgvector**, alongside metadata (document type, date, equipment tag) that allows hybrid search later — semantic similarity *filtered* by structured criteria.

**Step 3d — Cross-Link.**
An Edge Function tags every vector row with the ID of its corresponding graph node, and vice versa. This is a small but critical step: it means a semantic search hit can jump straight into graph traversal, and a graph node can pull its full original source text. Without this link, the graph and the vector store would be two disconnected systems instead of one unified knowledge base.

*→ Output of Phase 3: a fact is now stored twice, in two different ways that reinforce each other — as a relationship, and as a meaning.*

---

### Phase 4 — Agentic Reasoning *(Architecture Layer 4)*

This phase is different from the first three: it doesn't run *because* a document arrived — it runs on its own schedule, continuously watching the knowledge base for patterns. This is what makes the platform proactive rather than purely reactive.

**Step 4a — Scheduled Scan.**
On a fixed interval, a **GitHub Actions cron job** wakes the **Lessons Learned agent**, which pulls every graph node added since its last run.

**Step 4b — Pattern Matching.**
Each new finding is compared against historical failure patterns already in the graph — via **LangGraph**-orchestrated Cypher queries against **Neo4j** — looking for matches on equipment class, symptom language, or recurring root causes.

**Step 4c — Decision Branch.**
- If the match is **above threshold**, the finding is escalated to the next step.
- If it's **below threshold**, the result is logged silently and no one is interrupted. This threshold matters: a system that alerts on everything trains people to ignore it.

**Step 4d — Proactive Push.**
For an escalated match, the LLM generates a plain-language warning describing the pattern, and **Firebase Cloud Messaging** pushes it directly to the relevant field technician's device — *before anyone asked*. This is the "knowledge cliff" problem in action: the pattern-recognition an experienced engineer would have done from memory is now happening automatically, continuously, against the full historical record instead of one person's recollection.

*Note: the other three agents — Expert Knowledge Copilot, Maintenance & RCA Agent, and Compliance Intelligence — live in this same layer and share the same orchestration runtime (LangGraph/CrewAI, Gemini Flash + Groq), but are triggered by a query rather than a schedule. The Copilot's full trigger-to-answer path is traced in Phase 5.*

---

## PATH B — From a Person's Question to a Grounded Answer

### Phase 5 — Query & Delivery *(Architecture Layer 5)*

This is the path a person actually experiences. It can happen completely independently of Phases 1–4 having just run — it queries whatever is already in the knowledge base, built up from every document ever ingested.

**Step 5a — Question Asked.**
A field technician opens the **PWA on their phone** (or an engineer uses the desktop React app) and asks something in plain language — e.g. *"Any known issues with Pump P-204?"* This becomes an API request.

**Step 5b — Auth Check.**
**Supabase Auth / Clerk** authenticates the request and scopes what the person is allowed to see. A technician might see operational and maintenance data; certain safety or regulatory records might be restricted to certified roles. This happens *before* any retrieval — access control is enforced at the door, not filtered after the fact.

**Step 5c — Hybrid Retrieval.**
The **FastAPI backend** does two things at once: a semantic search against **pgvector** to find the most relevant text chunks, and a graph traversal in **Neo4j** to pull the full relationship context around the matched equipment — prior findings, related work orders, connected components. This is the payoff of Phase 3's cross-linking: the system isn't just finding a paragraph that mentions "Pump P-204," it's finding *everything connected to* Pump P-204.

**Step 5d — Grounded Generation.**
That retrieved context — chunks plus graph relationships — is handed to the LLM (**Groq** for speed, or **Gemini Flash**) with an instruction to answer *only* from what was retrieved, and to cite where each claim came from. This is what keeps the answer trustworthy: the model isn't reasoning from general knowledge about pumps, it's reasoning from this specific plant's actual history.

**Step 5e — Delivery.**
The answer is displayed in the PWA with citations that link directly back to the original source document in Supabase Storage. The technician isn't asked to trust the answer blindly — they can tap through and verify it against the real inspection report or work order it came from.

*→ Output of Phase 5: a specific, sourced answer, delivered on whatever device the person happens to be holding.*

---

## Why the Two Paths Matter Together

The reason this is a "Unified Asset & Operations Brain" and not just a chatbot bolted onto a filing cabinet is that **Path A never stops running while Path B is being used.** Every new inspection report, work order, or drawing that gets ingested through Phase 1–3 immediately becomes part of what Phase 5 can retrieve from, and immediately becomes something Phase 4 can pattern-match against for the *next* technician's proactive warning.

In short:

- **Documents go in continuously** (Path A) → the knowledge graph and vector store get richer with every upload.
- **Answers come out on demand** (Path B, Step 5) → grounded in everything that's ever gone in.
- **Warnings come out unprompted** (Path A, Phase 4) → because the system is always comparing new information against old patterns, even when nobody's asking a question.

That continuous loop — ingest, understand, connect, watch, answer — is what the five-layer architecture and the phase-by-phase workflow are both describing from two different angles: the architecture diagram shows *what the system is made of*; the workflow diagram shows *what it actually does, in order*, every time a document arrives or a question gets asked.
