# Context-Aware Multi-Agent AI System for Dynamic Contract Risk Prediction — System Walkthrough

This document traces how the platform behaves in practice — from the moment a new contract is uploaded, through knowledge graph structuring, to the execution of a multi-agent simulation that predicts cascading enterprise risks. Each phase below has a Mermaid diagram directly beneath its heading, with explicit **INPUT** and **OUTPUT** nodes so you can see at a glance what enters and leaves that phase.

There are **two entry points**:

1. **The Ingestion Path** — a draft contract is uploaded, parsed, and mapped into the enterprise ecosystem (Phases 1–3).
2. **The Simulation Path** — specialized AI agents simulate the real-world operational and financial impacts of the contract before execution (Phases 4–5).

---

## Overview — How the Two Paths Connect

```mermaid
flowchart LR
    IN1([INPUT<br/>Draft Contract PDF]):::io --> P1[Phase 1<br/>Ingestion & Parsing]
    P1 --> P2[Phase 2<br/>Clause Extraction]
    P2 --> P3[Phase 3<br/>Graph Structuring]
    P3 -.updates ecosystem.-> P4

    IN2([INPUT<br/>User 'Simulate' Action]):::io --> P4[Phase 4<br/>Multi-Agent Simulation]
    P4 --> P5[Phase 5<br/>Negotiation & UI Rendering]
    P5 --> OUT([OUTPUT<br/>Interactive Risk Dashboard]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#2f5875,color:#cfe3ee
```

---

## PATH A — From Raw Contract to Enterprise Knowledge Graph

### Phase 1 — Ingestion & Parsing *(Architecture Layer 1)*

```mermaid
flowchart TD
    IN([INPUT<br/>Contract PDF / Docx]):::io --> A[1a. File Upload<br/><i>Next.js Frontend</i>]
    A --> B[1b. Storage & Trigger<br/><i>Supabase Storage</i>]
    B --> C[1c. Text Extraction<br/><i>FastAPI / LlamaParse</i>]
    C --> OUT([OUTPUT<br/>Raw Text Chunks]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#3f6fa8,color:#cfe3ee
```

**Trigger:** A legal or procurement officer uploads a draft supplier contract into the portal.

**Step 1a — File Upload.** The **Next.js** frontend captures the document and passes it to the backend architecture.
**Step 1b — Storage.** The file is securely stored in **Supabase Storage**.
**Step 1c — Text Extraction.** A **FastAPI** backend (hosted on Render) utilizes LlamaParse or basic PDF extractors to strip out boilerplate and chunk the text into logical sections, preserving tables and SLA metrics.

---

### Phase 2 — LLM Clause & Entity Extraction *(Architecture Layer 2)*

```mermaid
flowchart TD
    IN([INPUT<br/>Raw Text Chunks]):::io --> D[2a. Context Prompting<br/><i>FastAPI Logic</i>]
    D --> E[2b. Entity Extraction<br/><i>Gemini 1.5 API</i>]
    E --> F[2c. JSON Validation<br/><i>Pydantic</i>]
    F --> OUT([OUTPUT<br/>Structured Contract<br/>JSON Payload]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#3f9d63,color:#cfe3ee
```

Here, unstructured legalese is translated into structured business data.

**Step 2a — Context Prompting.** The backend constructs prompts designed to isolate specific entities: supplier names, delivery timelines, penalty percentages, and governing jurisdictions.
**Step 2b — Entity Extraction.** The **Google Gemini 1.5 API** processes the chunks in JSON mode. Its massive context window allows it to process the entire contract simultaneously without losing references.
**Step 2c — JSON Validation.** **Pydantic** validates that the LLM output conforms to the required data schema before passing it to the database.

---

### Phase 3 — Knowledge Graph Structuring *(Architecture Layer 3)*

```mermaid
flowchart TD
    IN([INPUT<br/>Structured Contract<br/>JSON Payload]):::io --> G[3a. Cypher Generation<br/><i>FastAPI</i>]
    G --> H[3b. Node/Edge MERGE<br/><i>Neo4j AuraDB</i>]
    H --> I[3c. Enterprise Linkage<br/><i>Graph Traversal</i>]
    I --> OUT([OUTPUT<br/>Updated Enterprise<br/>Knowledge Graph]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#9b5fc9,color:#cfe3ee
```

This is the key differentiator from standard contract AI. The contract is not analyzed in a vacuum; it is wired into the company's nervous system.

**Step 3a — Cypher Generation.** The JSON data is translated into Cypher queries.
**Step 3b — Node/Edge MERGE.** Nodes (e.g., "Supplier A", "Delivery Clause") and Edges (e.g., "TRIGGERS_PENALTY") are written into **Neo4j AuraDB**.
**Step 3c — Enterprise Linkage.** The database automatically links this new contract to existing nodes—for instance, linking "Supplier A's" raw material delivery to "Customer B's" finalized product commitments.

---

## PATH B — Multi-Agent Risk Simulation

### Phase 4 — Multi-Agent Reasoning *(Architecture Layer 4)*

```mermaid
flowchart TD
    IN([INPUT<br/>User 'Simulate' Trigger]):::io --> J[4a. Orchestration<br/><i>LangGraph</i>]
    J --> K[4b. Graph Context Queries<br/><i>Gemini + Neo4j</i>]
    K --> L{4c. Agent Roles}
    L --> M[Legal Agent]
    L --> N[Financial Agent]
    L --> O[Operations Agent]
    M & N & O --> P[4d. Aggregate Findings]
    P --> OUT([OUTPUT<br/>Multi-Domain Risk<br/>Scores & Impacts]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#c96a3f,color:#cfe3ee
```

**Step 4a — Orchestration.** A user clicks "Simulate Risk." **LangGraph** initializes a multi-agent workflow.
**Step 4b — Context Queries.** The orchestrator pulls the cascading graph data from Neo4j (e.g., "Supplier A is 3 hops away from Customer B").
**Step 4c — Agent Roles.** 
*   **Legal Agent** reviews standard compliance.
*   **Operations Agent** sees that a 10-day delay allowed in this contract will cause a stockout on the assembly line.
*   **Financial Agent** calculates that the assembly line stockout will trigger a $50,000 SLA penalty with Customer B.
**Step 4d — Aggregate Findings.** LangGraph compiles the diverse perspectives into a single unified risk report.

---

### Phase 5 — Negotiation & UI Rendering *(Architecture Layer 5)*

```mermaid
flowchart TD
    IN([INPUT<br/>Multi-Domain Risk<br/>Scores & Impacts]):::io --> Q[5a. Alternative Generation<br/><i>Negotiation Agent</i>]
    Q --> R[5b. Payload Formatting<br/><i>FastAPI</i>]
    R --> S[5c. UI Rendering<br/><i>Next.js + React Flow</i>]
    S --> OUT([OUTPUT<br/>Interactive 'What-If'<br/>Dashboard]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#3fa3a3,color:#cfe3ee
```

**Step 5a — Alternative Generation.** Before showing the user the problem, a **Negotiation Agent** (powered by Gemini) drafts alternative clause text (e.g., tightening the delivery window to 5 days) to neutralize the cascading risk.
**Step 5b/c — Rendering.** The backend sends the full simulation to **Next.js**. The user sees a visual network graph of how the risk cascades through the enterprise, accompanied by plain-English rewrite suggestions to fix it.

---

## Why the Two Paths Matter Together

```mermaid
flowchart LR
    subgraph LOOP[" "]
        direction LR
        DOC[Contract<br/>Ingestion] --> KG[(Enterprise<br/>Knowledge Graph)]
        KG --> SIM[Multi-Agent<br/>Simulation]
        SIM --> DASH[User Negotiates<br/>Safer Contract]
        DASH -.prevents future risk.-> KG
    end
    classDef default fill:#0f2033,stroke:#4fd8e8,color:#cfe3ee
    classDef kb fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    class KG kb
```

Traditional tools just check if a clause is missing. This architecture predicts the future.
- **Path A builds the interconnected reality** of the business, ensuring no contract exists in a vacuum.
- **Path B unleashes specialized AI agents** onto that reality, allowing them to debate and calculate cascading consequences before a human signs a flawed document, saving millions in hidden operational liabilities.
