# AI for Retail Intelligence: Reverse Logistics Optimization Brain — System Walkthrough

This document traces how the platform behaves in practice — from the moment a user begins a checkout process, through real-time risk prediction, to the moment an intervention is served and the system learns from the outcome. Each phase below has a Mermaid diagram directly beneath its heading, with explicit **INPUT** and **OUTPUT** nodes so you can see at a glance what enters and leaves that phase.

There are **two entry points**:

1. **The Transaction Path** — a user attempts to buy something, and the system must predict and mitigate return risk (Phases 1–3).

2. **The Operator Path** — a retail manager reviews prevented returns and updates dynamic policies (Phase 4).

Both paths share the same underlying database and model state, ensuring that every intervention makes the next prediction smarter.

## Overview — How the Two Paths Connect

```mermaid
flowchart LR
    IN([INPUT<br/>User Checkout Intent]):::io --> P1[Phase 1<br/>Session Ingestion]
    P1 --> P2[Phase 2<br/>Risk Prediction]
    P2 --> P3[Phase 3<br/>Dynamic Intervention]
    P3 --> OUT1([OUTPUT<br/>Modified Checkout UI]):::io

    IN2([INPUT<br/>Operator Query]):::io --> P4[Phase 4<br/>Analytics & RL Update]
    P3 -.logs outcome.-> P4
    P4 --> OUT2([OUTPUT<br/>Salvaged Margin Dashboard]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#2f5875,color:#cfe3ee
```

## PATH A — From Checkout Intent to Dynamic Intervention

### Phase 1 — Session Ingestion *(Architecture Layer 1)*

```mermaid
flowchart TD
    IN([INPUT<br/>Cart Data +<br/>Session ID]):::io --> A[1a. Checkout Trigger<br/><i>Next.js Frontend</i>]
    A --> B[1b. API Router<br/><i>FastAPI Backend</i>]
    B --> C[1c. History Fetch<br/><i>Supabase PostgreSQL</i>]
    C --> OUT([OUTPUT<br/>Aggregated User<br/>Feature Vector]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#3f6fa8,color:#cfe3ee
```

**Trigger:** A shopper clicks "Proceed to Checkout" with a cart containing multiple sizes of the same jacket.

**Step 1a — Checkout Trigger.** The **Next.js Frontend** intercepts the standard checkout flow, bundling the cart contents, session dwell time, and device info into a JSON payload.

**Step 1b — API Router.** The payload hits the **FastAPI** backend hosted on Render. This acts as the orchestration layer for the transaction.

**Step 1c — History Fetch.** The backend queries **Supabase (PostgreSQL)** for the user's historical purchase and return rates. This merges historical behavior with real-time session data to create a complete feature vector for the ML model.

### Phase 2 — Risk Prediction *(Architecture Layer 2)*

```mermaid
flowchart TD
    IN([INPUT<br/>Aggregated User<br/>Feature Vector]):::io --> D[2a. Classification Model<br/><i>LightGBM / XGBoost</i>]
    D --> E{2b. Return Risk<br/>Threshold?}
    E -->|Low Risk| F[2c. Standard Flow<br/><i>Pass-through</i>]
    E -->|High Risk| G[2d. RL Agent Trigger<br/><i>Ray RLlib / Custom Agent</i>]
    F --> OUT1([OUTPUT<br/>Normal Checkout]):::io
    G --> OUT2([OUTPUT<br/>Selected Mitigation<br/>Strategy]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#3f9d63,color:#cfe3ee
```

This is where historical data transforms into actionable foresight, happening in milliseconds before the UI loads.

**Step 2a — Classification Model.** The feature vector is passed into a lightweight **LightGBM** model, which calculates a percentage probability that items in the cart will be returned.

**Step 2b — Risk Threshold.** The system evaluates the probability against the retailer's current margin threshold.

**Step 2c — Standard Flow.** If the risk is low, the system does nothing, allowing the user to check out normally without friction.

**Step 2d — RL Agent Trigger.** If the risk is high (e.g., >75%), a Reinforcement Learning agent selects the optimal intervention (e.g., Non-refundable discount, forced sizing chart, warning).

### Phase 3 — Dynamic Intervention *(Architecture Layer 3)*

```mermaid
flowchart TD
    IN([INPUT<br/>Selected Mitigation<br/>Strategy]):::io --> H[3a. Prompt Generation<br/><i>FastAPI Context Builder</i>]
    H --> I[3b. Dynamic Copy<br/><i>Google Gemini API</i>]
    I --> J[3c. UI Injection<br/><i>Next.js State Update</i>]
    J --> OUT([OUTPUT<br/>Modified Checkout UI<br/>presented to user]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#9b5fc9,color:#cfe3ee
```

This layer dynamically alters the user experience based on the AI's decision, aiming to change behavior rather than just predict it.

**Step 3a — Prompt Generation.** Based on the RL agent's strategy, the backend formats a contextual prompt (e.g., "Create a 1-sentence urgent offer for a 15% discount if the user makes this sale final").

**Step 3b — Dynamic Copy.** The **Google Gemini API** generates persuasive, personalized copy on the fly, preventing the UI from feeling robotic or canned.

**Step 3c — UI Injection.** The **Next.js** frontend receives the payload and renders a localized modal or friction point, intercepting the user before payment is processed.

## PATH B — The Operator Analytics Path

### Phase 4 — Analytics & RL Update *(Architecture Layer 4)*

```mermaid
flowchart TD
    IN([INPUT<br/>Transaction Outcome<br/>Logs]):::io --> K[4a. Database Write<br/><i>Supabase</i>]
    K --> L[4b. Agent Reward Calc<br/><i>Background Worker</i>]
    L --> M[4c. Dashboard Render<br/><i>Streamlit / Next.js Admin</i>]
    M --> OUT([OUTPUT<br/>Salvaged Margin KPIs]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#c96a3f,color:#cfe3ee
```

This path runs asynchronously, ensuring retail managers have visibility into the AI's performance and the AI learns from its successes and failures.

**Step 4a — Database Write.** Whether the user accepted the "Final Sale" discount, abandoned the cart, or completed a normal checkout, the result is logged in **Supabase**.

**Step 4b — Agent Reward Calc.** A background worker calculates the reward function (e.g., profit margin saved minus discount cost) and updates the Reinforcement Learning agent's weights for future decisions.

**Step 4c — Dashboard Render.** A retail operator opens the Admin Dashboard, which queries Supabase to display real-time metrics on prevented returns and total margin salvaged by the AI.

## Why the Two Paths Matter Together

```mermaid
flowchart LR
    subgraph LOOP[" "]
        direction LR
        TRANS[Live Transactions<br/>Intercepted] --> DB[(Behavior & Outcome<br/>Ledger)]
        DB --> MODEL[RL Model Weights<br/>Updated]
        DB --> DASH[Operator Dashboard<br/>Monitors ROI]
        MODEL -.smarter interventions.-> TRANS
    end
    classDef default fill:#0f2033,stroke:#4fd8e8,color:#cfe3ee
    classDef kb fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    class DB kb
```

The system is a closed-loop intelligence engine.
- **Transactions are intercepted (Path A)** to protect immediate profit margins.
- **Outcomes are logged and analyzed (Path B)**, directly training the Reinforcement Learning agent.
- Every accepted or rejected intervention makes the system more accurate at pricing risk for the *next* shopper, while providing operators with a mathematically proven ROI dashboard.
