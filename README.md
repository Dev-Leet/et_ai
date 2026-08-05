graph TD
    IN(["📥 INPUT: User Question"]) --> API(["FastAPI Backend"])
    API --> Auth{"Supabase Auth"}
    
    Auth -->|Access Granted| R(["Hybrid Retrieval"])
    
    R -->|Semantic Search| V(["pgvector"])
    R -->|Context Search| N(["Neo4j"])
    
    V --> Context(["Combined Context"])
    N --> Context
    
    Context --> LLM(["Groq / Gemini Flash"])
    LLM --> OUT(["📤 OUTPUT: Answer + Source Citations"])

    style IN fill:#e1f5fe,stroke:#0288d1
    style OUT fill:#e8f5e9,stroke:#2e7d32
