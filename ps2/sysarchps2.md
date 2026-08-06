# AI for Supply Chain Resilience: Autonomous Digital Twin — System Walkthrough

This document traces how the platform behaves in practice — from the continuous ingestion of supply chain data, through the detection of a disruption, to the autonomous calculation of a new optimal routing path. Each phase below has a Mermaid diagram directly beneath its heading, with explicit **INPUT** and **OUTPUT** nodes so you can see at a glance what enters and leaves that phase.

There are **two entry points**:

1. **The Environment Path** — scheduled jobs ingest global data and update the network baseline (Phases 1–2).
2. **The Disruption Path** — a node fails, and the system autonomously rebalances the logistics network (Phases 3–4).

## Overview — How the Two Paths Connect

```mermaid
flowchart LR
    IN1([INPUT<br/>Macro/Transit Data]):::io --> P1[Phase 1<br/>Continuous Ingestion]
    P1 --> P2[Phase 2<br/>Graph State Update]
    P2 -.live digital twin.-> P3

    IN2([INPUT<br/>Simulated Disruption]):::io --> P3[Phase 3<br/>Optimization Engine]
    P3 --> P4[Phase 4<br/>UI Visualization]
    P4 --> OUT([OUTPUT<br/>Rerouted Supply Map]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#2f5875,color:#cfe3ee
```

## PATH A — From Global Data to Digital Twin Baseline

### Phase 1 — Continuous Ingestion *(Architecture Layer 1)*

```mermaid
flowchart TD
    IN([INPUT<br/>Time-Series Data<br/>APIs / CSVs]):::io --> A[1a. Cron Trigger<br/><i>GitHub Actions</i>]
    A --> B[1b. Data Aggregation<br/><i>FastAPI Data Pipeline</i>]
    B --> C[1c. Demand Forecasting<br/><i>PyTorch LSTM / Prophet</i>]
    C --> OUT([OUTPUT<br/>Predicted Node<br/>Demand Volumes]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#3f6fa8,color:#cfe3ee
```

**Trigger:** A scheduled cron job runs daily to update the baseline reality of the global supply chain.

**Step 1a — Cron Trigger.** **GitHub Actions** fires a scheduled event, pinging the backend to initiate the daily data pull without needing a complex tool like Airflow.

**Step 1b — Data Aggregation.** The **FastAPI** backend fetches mock transit delays, historical sales data, and macroeconomic indicators.

**Step 1c — Demand Forecasting.** The data is fed into a lightweight **PyTorch** or Prophet time-series model to predict localized demand at various network nodes for the upcoming week.

### Phase 2 — Graph State Update *(Architecture Layer 2)*

```mermaid
flowchart TD
    IN([INPUT<br/>Predicted Node<br/>Demand Volumes]):::io --> D[2a. Cypher Translation<br/><i>FastAPI Logic</i>]
    D --> E[2b. Graph Write<br/><i>Neo4j AuraDB</i>]
    E --> F[2c. Relational Logging<br/><i>Supabase</i>]
    E --> OUT([OUTPUT<br/>Updated Network<br/>Graph State]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#3f9d63,color:#cfe3ee
```

This phase translates numerical predictions into physical relationships in the Digital Twin.

**Step 2a — Cypher Translation.** The backend maps demand forecasts and transit times into Cypher queries (e.g., updating edge weights representing transit costs).

**Step 2b — Graph Write.** **Neo4j AuraDB** updates its nodes (warehouses, ports) and edges (shipping routes). The database now perfectly mirrors current global conditions.

**Step 2c — Relational Logging.** A simple log is written to **Supabase** to maintain an audit trail of daily network changes.

## PATH B — The Disruption & Rebalancing Path

### Phase 3 — Optimization Engine *(Architecture Layer 3)*

```mermaid
flowchart TD
    IN([INPUT<br/>Node Failure Event<br/>e.g., Port Closure]):::io --> G[3a. Disruption Inject<br/><i>React.js / API</i>]
    G --> H[3b. Network Pull<br/><i>Neo4j AuraDB</i>]
    H --> I[3c. Linear Optimization<br/><i>SciPy / NetworkX</i>]
    I --> OUT([OUTPUT<br/>Mathematically Optimal<br/>Reroute Paths]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#9b5fc9,color:#cfe3ee
```

This is the core "AI Solver" phase. It turns the system from a passive dashboard into an autonomous agent.

**Step 3a — Disruption Inject.** An operator clicks "Simulate Strike" on a specific port node in the UI, sending a disruption payload to the API.

**Step 3b — Network Pull.** The backend immediately queries **Neo4j** for the entire current graph topology, minus the offline node.

**Step 3c — Linear Optimization.** **SciPy** and **NetworkX** run a minimum-cost maximum-flow algorithm (or linear programming solver) against the graph, mathematically balancing the cheapest freight costs against the urgency of the forecasted demand.

### Phase 4 — UI Visualization *(Architecture Layer 4)*

```mermaid
flowchart TD
    IN([INPUT<br/>Optimal Reroute<br/>Paths]):::io --> J[4a. State Sync<br/><i>Neo4j Update</i>]
    J --> K[4b. Payload Delivery<br/><i>FastAPI → React</i>]
    K --> L[4c. Map Rendering<br/><i>React Flow / D3.js</i>]
    L --> OUT([OUTPUT<br/>Visualized Recovery<br/>Dashboard]):::io

    classDef io fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    classDef default fill:#0f2033,stroke:#c96a3f,color:#cfe3ee
```

**Step 4a — State Sync.** The new optimal routes are written back to **Neo4j** as the new active truth.

**Step 4b — Payload Delivery.** The backend sends a JSON payload containing the new edges, costs, and delays to the frontend.

**Step 4c — Map Rendering.** The **React** frontend (using React Flow or Deck.gl) animates the rerouting, visually showing the operator exactly how freight has been shifted to avoid the disruption, alongside the financial impact.

## Why the Two Paths Matter Together

```mermaid
flowchart LR
    subgraph LOOP[" "]
        direction LR
        DATA[Daily Demand &<br/>Transit Updates] --> TWIN[(Live Graph<br/>Digital Twin)]
        TWIN --> SOLVER[Disruption Triggered<br/>SciPy Solver]
        SOLVER --> DASH[Operator Re-routes<br/>Network]
        SOLVER -.updates baseline.-> TWIN
    end
    classDef default fill:#0f2033,stroke:#4fd8e8,color:#cfe3ee
    classDef kb fill:#0d1b2b,stroke:#e8a13e,stroke-width:2px,color:#eaf3fa
    class TWIN kb
```

A digital twin is useless if it's out of date, and optimization is useless if it runs on historical averages. 
- **Path A ensures the graph is always alive**, reflecting current macroeconomic realities.
- **Path B allows operators to instantly stress-test that reality.** 
Because the solver interacts directly with the live graph, the rerouting suggestions are mathematically actionable right now, not just theoretical planning exercises.
