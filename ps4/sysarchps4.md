# Explainable AI for Software QA: Defect Prediction Pipeline — System Walkthrough

This document traces how the platform behaves in practice — from a developer uploading code, through ML prediction and AI explanation, to the delivery of actionable refactoring advice. Each phase below has a Mermaid diagram directly beneath its heading, with explicit **INPUT** and **OUTPUT** nodes so you can see at a glance what enters and leaves that phase.

There are **two entry points**:

1. **The Diagnostic Path** — code is analyzed, scored, and mathematically explained (Phases 1–2).
2. **The Generative Path** — math is translated into plain-English advice via LLMs and delivered to the developer (Phases 3–4).

## Overview — How the Two Paths Connect

```mermaid
flowchart LR
    IN1([INPUT<br/>Code Metrics JSON]):::io --> P1[Phase 1<br/>ML Prediction]
    P1 --> P2[Phase 2<br/>Explainability XAI]
    P2 -.SHAP values.-> P3

    P3[Phase 3<br/>LLM Translation] --> P4[Phase 4<br/>Developer Portal]
    P4 --> OUT([OUTPUT<br/>Actionable Refactoring<br/>Insights]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#2f5875,color:#cfe3ee
```

## PATH A — From Code to Mathematical Diagnostics

### Phase 1 — ML Prediction *(Architecture Layer 1)*

```mermaid
flowchart TD
    IN([INPUT<br/>Code Metrics<br/>Complexity/LOC]):::io --> A[1a. File Upload<br/><i>Next.js UI</i>]
    A --> B[1b. API Ingestion<br/><i>FastAPI</i>]
    B --> C[1c. Classification<br/><i>XGBoost Classifier</i>]
    C --> OUT([OUTPUT<br/>Defect Probability %]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#3f6fa8,color:#cfe3ee
```

**Trigger:** A developer or automated CI/CD pipeline pushes static code metrics (Lines of Code, Cyclomatic Complexity, Halstead Metrics) to the portal.

**Step 1a — File Upload.** The **Next.js** frontend captures the payload and sends it to the backend.

**Step 1b — API Ingestion.** The **FastAPI** backend (hosted on Render) receives the payload and formats it into a Pandas DataFrame.

**Step 1c — Classification.** The structured data is passed through a pre-trained **XGBoost** model which outputs a binary classification (Faulty / Clean) and a confidence probability (e.g., 88% chance of defect).

### Phase 2 — Explainability (XAI) *(Architecture Layer 2)*

```mermaid
flowchart TD
    IN([INPUT<br/>Defect Probability %]):::io --> D{2a. High Risk?}
    D -->|No| E[Pass to UI]
    D -->|Yes| F[2b. SHAP Explainer<br/><i>shap Python Lib</i>]
    F --> G[2c. Feature Isolation<br/><i>FastAPI Logic</i>]
    E --> OUT1([OUTPUT<br/>Clean Status]):::io
    G --> OUT2([OUTPUT<br/>Top Contributing<br/>SHAP Values]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#3f9d63,color:#cfe3ee
```

This phase solves the "Black Box" problem of ML, answering *why* the model predicted a failure.

**Step 2a — Risk Check.** The system evaluates the model output. If the code is predicted "Clean," it bypasses heavy computation.

**Step 2b — SHAP Explainer.** For high-risk code, the data is passed to the **SHAP (SHapley Additive exPlanations)** library. SHAP calculates exactly how much each specific metric (e.g., complexity) contributed to the final 88% risk score.

**Step 2c — Feature Isolation.** The backend isolates the top 3 mathematical reasons for the predicted failure.

## PATH B — From Math to Actionable Insights

### Phase 3 — LLM Translation *(Architecture Layer 3)*

```mermaid
flowchart TD
    IN([INPUT<br/>Top Contributing<br/>SHAP Values]):::io --> H[3a. Prompt Formatting<br/><i>LangChain / FastAPI</i>]
    H --> I[3b. GenAI Query<br/><i>Groq API - Llama 3</i>]
    I --> J[3c. DB Storage<br/><i>Supabase</i>]
    I --> OUT([OUTPUT<br/>Plain-English<br/>Refactor Advice]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#9b5fc9,color:#cfe3ee
```

Developers don't want raw SHAP arrays; they want solutions. This phase translates data science into software engineering.

**Step 3a — Prompt Formatting.** The backend constructs a prompt: *"You are a senior dev. The ML model flagged this function because Cyclomatic Complexity is +4.5 over the baseline. Explain to a junior dev how to fix this."*

**Step 3b — GenAI Query.** The prompt is sent to the ultra-fast **Groq API** running open-source Llama 3, which generates actionable, contextual refactoring advice.

**Step 3c — DB Storage.** The original metrics, prediction score, and LLM advice are saved to **Supabase** for historical QA tracking.

### Phase 4 — Developer Portal *(Architecture Layer 4)*

```mermaid
flowchart TD
    IN([INPUT<br/>Plain-English Advice +<br/>Prediction Data]):::io --> K[4a. Payload Delivery<br/><i>FastAPI → Next.js</i>]
    K --> L[4b. Data Visualization<br/><i>Recharts / Chart.js</i>]
    L --> M[4c. Insight Display<br/><i>UI Render</i>]
    M --> OUT([OUTPUT<br/>Dev Dashboard UI]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#c96a3f,color:#cfe3ee
```

**Step 4a — Payload Delivery.** FastAPI returns the complete analysis package to the client.

**Step 4b — Data Visualization.** The **Next.js** UI uses simple charting libraries (like Recharts) to render the SHAP values visually as a waterfall or bar chart.

**Step 4c — Insight Display.** The plain-English advice is displayed alongside the charts, giving the developer a complete, understandable diagnostic report.

## Why the Two Paths Matter Together

```mermaid
flowchart LR
    subgraph LOOP[" "]
        direction LR
        CODE[Raw Code Metrics] --> ML[(XGBoost ML<br/>Predicts Defect)]
        ML --> SHAP[SHAP Isolates<br/>Mathematical Cause]
        SHAP --> LLM[LLM Translates Math<br/>to Dev Advice]
        LLM -.developer fixes code.-> CODE
    end
    classDef default fill:#0f2033,stroke:#4fd8e8,color:#cfe3ee
    classDef kb fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    class ML kb
```

Without Phase 2 (SHAP), the system is just a black box yelling "Error!" at a developer. Without Phase 3 (LLM), the system is just a math tool requiring data science knowledge to interpret. 
- **Path A handles the rigorous statistical prediction.**
- **Path B handles the human-in-the-loop communication.** 
Together, they form a complete GenAI MLOps pipeline that not only finds bugs before they happen but actually coaches the developer on how to fix them.
