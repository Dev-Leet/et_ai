---

## **Layer 1 — Data Ingestion**

| Component | Old (Heavy) | New (Student-Friendly) | Justification |
| ----- | ----- | ----- | ----- |
| File/object storage | MinIO | **Supabase Storage** (or Firebase Storage) | S3-like storage bundled free with Supabase (1GB free, cheap beyond) — no server to run, just an API. |
| Structured metadata \+ relational data | Self-hosted PostgreSQL | **Supabase (managed Postgres)** | Free tier gives a full Postgres instance with a dashboard, REST API auto-generated, and no DB admin overhead. |
| Event/message queue | Apache Kafka | **Direct API calls \+ Supabase Edge Functions**, or **Upstash Kafka (serverless, free tier)** if you want real async processing | Kafka needs a cluster to manage. For a prototype, a simple "upload → trigger function → process" flow is enough. Upstash gives serverless Kafka-compatible queuing if you want to keep the async pattern without hosting anything. |
| Ingestion/dataflow orchestration | Apache NiFi | **n8n Cloud (free tier)** or simple Python scripts triggered by Supabase Edge Functions | NiFi is a heavyweight enterprise tool. n8n's free cloud tier gives visual workflow building without hosting infrastructure. For a project this size, even a scheduled Python script (via GitHub Actions cron) is often enough. |
| Email ingestion | Apache James | **Gmail API (free quota)** or IMAP via Python `imaplib` | No mail server needed — just pull from a Gmail account via API, well within free quota for a student volume of emails. |

---

## 

## **Layer 2 — Processing (OCR / CV / NLP)**

| Component | Old (Heavy) | New (Student-Friendly) | Justification |
| ----- | ----- | ----- | ----- |
| General OCR | PaddleOCR (self-hosted, GPU-friendly but heavy setup) | **Google Cloud Vision API (free tier: 1,000 units/month)** or keep **PaddleOCR but run it on Google Colab free GPU** | Cloud Vision's free tier is generous for a demo corpus and needs zero setup. If you want to stay "open-source," Colab lets you run PaddleOCR without needing your own GPU — but Cloud Vision is simpler to keep running reliably. |
| Document layout & table extraction | Docling / LayoutParser (local) | **LlamaParse (LlamaIndex's hosted parser, free tier: 1,000 pages/day)** | Purpose-built for exactly this (PDFs → structured Markdown), generous free daily quota, zero local compute — a strong upgrade over self-hosting Docling for a prototype. |
| P\&ID / drawing symbol detection | OpenCV \+ Detectron2 (needs training \+ GPU) | **Google Cloud Vision API (object localization) as a baseline**, or skip fine-tuned detection entirely and demo on a small hand-labeled sample using OpenCV alone | Detectron2 fine-tuning needs labeled data and GPU time you likely don't have. For a project demo, scope this down: OpenCV for line/contour detection \+ a small rule-based tag reader is defensible, and you can be upfront in your report that production would need a fine-tuned CV model. |
| Handwriting recognition | TrOCR (local, HuggingFace) | **Google Cloud Vision API** (handles handwriting in same free quota) | One API covers both printed and handwritten OCR — avoids running a second local transformer model. |
| Entity extraction / NER | spaCy (local) \+ local LLM | **Gemini 1.5 Flash API** for extraction (structured JSON output mode) | Flash's free tier (15 RPM / 1M tokens/day as of last check — verify current limits) is enough for prototype-scale entity extraction, and skips the need to fine-tune or even run spaCy locally. Use Gemini's JSON mode to get structured entities directly. |
| LLM inference (all reasoning tasks) | Ollama / vLLM (local GPU) | **Gemini 1.5 Flash API** (primary) \+ **Groq API** (for fast Llama 3.1/3.3 inference, free tier) | Gemini Flash: strong free quota, good at structured extraction and long context (useful for big documents). Groq: extremely fast inference on open-weight models (Llama 3.3 70B) — good for demoing low-latency chat responses. Having both also lets you show comparative reasoning in your report. GitHub Student Pack also unlocks OpenAI/Azure credits worth checking. |
| Chunking | LangChain/LlamaIndex splitters | **Keep as-is** | These are lightweight Python libraries, not infra — no change needed, they already run fine locally. |

---

## **Layer 3 — Knowledge Representation**

| Component | Old (Heavy) | New (Student-Friendly) | Justification |
| ----- | ----- | ----- | ----- |
| Knowledge graph | Neo4j Community (self-hosted) | **Neo4j AuraDB Free Tier** | Fully managed, free tier gives \~200k nodes/relationships — plenty for a prototype corpus. No install, browser-based Cypher console included. |
| Vector database | Qdrant/Milvus (self-hosted) | **Supabase pgvector** (if you're already on Supabase for storage/auth — keeps everything in one place) or **Pinecone free tier** (simpler dedicated vector API) | pgvector inside Supabase avoids managing a second service and free-tier Postgres storage covers a prototype-sized corpus. Pinecone is the alternative if you want a purpose-built vector API with better ANN performance at scale — either is defensible; pgvector wins on simplicity since you're already using Supabase. |
| Embedding model | BAAI/bge-large (local) | **Gemini Embedding API (free tier)** or **Cohere Embed (free trial tier)** | Avoids running a local embedding model entirely — one less thing to host, and keeps embedding \+ generation on the same API surface if using Gemini throughout. |
| Graph-vector sync | Custom Python sync service | **Keep the pattern, but simplify**: a Supabase Edge Function triggered on new row insert calls both AuraDB and pgvector | Same logic as before, just implemented as a serverless function instead of a standalone service — no server to keep alive. |

---

## **Layer 4 — AI / Agent Orchestration**

| Component | Old (Heavy) | New (Student-Friendly) | Justification |
| ----- | ----- | ----- | ----- |
| RAG framework | LlamaIndex / LangChain | **Keep both — no change** | These are just Python libraries; they don't require hosting. Fine to keep running locally or in a lightweight backend (Vercel/Render free tier). |
| Multi-agent orchestration | LangGraph / CrewAI | **Keep — no change**, but run on a hosted backend instead of your laptop | Same reasoning — it's code, not infra. Deploy the FastAPI backend (see Layer 5\) to **Render free tier** or **Railway free tier** so it's not dependent on your laptop being on. |
| LLM inference | vLLM (self-hosted) | **Gemini 1.5 Flash / Groq** (already covered above) | Same as Layer 2 — one consistent LLM API layer used across extraction, RCA reasoning, and chat. |
| Workflow scheduling | Apache Airflow | **GitHub Actions (free tier, cron schedule)** or **Supabase Cron (pg\_cron)** | Airflow needs a persistent scheduler process. GitHub Actions gives free scheduled runs (e.g., nightly compliance scan) with zero hosting — very common for student projects. |
| Compliance rule engine | Drools (self-hosted Java rule engine) | **Simple Python rules (JSON/YAML-defined) \+ Gemini for narrative generation** | Drools requires a JVM environment and is overkill for a prototype's regulatory rule set. A small Python rules table (e.g., "inspection interval by equipment class") covers the same deterministic logic without a new dependency. |
| Evaluation/guardrails | RAGAS \+ Guardrails AI | **Keep both — no change** | Lightweight Python libraries, no hosting cost, genuinely useful for your evaluation section. |

---

## **Layer 5 — User Interfaces**

| Component | Old (Heavy) | New (Student-Friendly) | Justification |
| ----- | ----- | ----- | ----- |
| Backend API | FastAPI (self-hosted) | **FastAPI, deployed on Render/Railway free tier** | Same framework, just hosted for free instead of run locally — makes your demo accessible via a public URL instead of only on your laptop. |
| Desktop web app | React \+ Tailwind | **Keep, deploy on Vercel/Netlify free tier** | Standard free hosting for React apps, gives you a live demo link for your presentation. |
| Mobile app | React Native / Flutter | **Keep** — but consider a **responsive web app (PWA)** instead if time-constrained | Building and testing a real mobile app (App Store/Play Store) adds overhead for a student timeline. A PWA gives the "mobile-first for technicians" experience without app store deployment. |
| Notifications | Firebase FCM / ntfy | **Firebase Cloud Messaging (free tier)** | Simplest free option now that self-hosting ntfy isn't a hard requirement anymore — Firebase is well-documented and free at student scale. |
| Graph visualization | Cytoscape.js / Neo4j Bloom | **Neo4j AuraDB's built-in browser** for backend exploration \+ **Cytoscape.js** embedded in your React app for the user-facing view | AuraDB's console is free and good for your own debugging; Cytoscape.js stays for the polished in-app visualization since it's just a JS library, not infra. |
| Auth | Keycloak | **Supabase Auth** (or Clerk if you want a more polished UI out of the box) | Since you're already on Supabase for storage/DB, Supabase Auth keeps everything in one dashboard, one free tier, one set of credentials. Clerk is the better pick specifically if you want pre-built, attractive login UI components for the demo. |

---

## **Full Audit: Anything Still Too Heavy?**

Going through the whole stack once more, here's what to watch:

| Item | Status | Note |
| ----- | ----- | ----- |
| Detectron2 for P\&ID symbol detection | ⚠️ Still heavy | Needs labeled training data \+ GPU fine-tuning time. For a prototype, scope this down (see Layer 2 note) or explicitly frame it as "future work" in your report rather than trying to fully implement it. |
| Running PaddleOCR locally as a fallback | ⚠️ Optional heavy | Only needed if you want to avoid Cloud Vision's free-tier limits during heavy testing. Google Colab (free GPU) is the workaround if you hit API quota walls. |
| React Native mobile app | ⚠️ Time-heavy, not compute-heavy | Not infra-expensive, but development/testing overhead is real for a solo/small-team final-year project. PWA is the pragmatic swap. |
| Everything else (Supabase, AuraDB, pgvector/Pinecone, Gemini, Groq, Render/Vercel, GitHub Actions) | ✅ Confirmed light | All have genuinely usable free tiers with no server management on your end. |

**One more consideration:** Since your LLM calls now go through external APIs (Gemini, Groq), any real safety-incident or regulatory data used in your demo should be **synthetic or anonymized** — free-tier API terms often allow data to be used for provider model improvement unless you're on a paid/enterprise agreement. Worth a line in your report acknowledging this trade-off versus the original on-prem design.

---

## **Updated Data Flow Summary (End-to-End)**

**Scenario: A new inspection report PDF is uploaded.**

1. **Ingestion** — File uploaded to **Supabase Storage**; a Supabase Edge Function triggers on insert.  
2. **Processing** — The Edge Function calls **LlamaParse** to extract structured text/tables, and **Google Cloud Vision** for any scanned/handwritten sections. Extracted text is sent to **Gemini 1.5 Flash** (JSON mode) to pull entities (equipment tag, inspector, findings, date).  
3. **Knowledge Representation** — Entities are written as nodes/edges to **Neo4j AuraDB**; text chunks are embedded via **Gemini Embedding API** and stored in **Supabase pgvector**, tagged with the AuraDB node ID.  
4. **Agent Orchestration** — A **GitHub Actions** nightly job runs the Lessons Learned agent (LangGraph, calling Gemini/Groq), which queries AuraDB for pattern matches against historical failures and writes a flag back to Supabase if a match is found.  
5. **User Interface** — A field technician opens the **PWA** on their phone, asks "Any known issues with Pump P-204?" — the **FastAPI backend (on Render)** runs the RAG query against pgvector \+ AuraDB, calls **Groq** for a fast response, and returns an answer with citations linking back to the original PDF in Supabase Storage.

---

## **1\. Detectron2 for P\&ID Symbol Detection — Simplified Workaround**

The core problem: Detectron2 needs (a) a labeled dataset of P\&ID symbols, (b) GPU time to fine-tune, and (c) MLOps overhead to serve the model. For a final-year project, you don't need production-grade detection — you need a **defensible, demo-able pipeline** that shows the concept works on a representative sample.

### **Simplified Architecture**

P\&ID Image/PDF

     ↓

\[Step 1\] Pre-processing (OpenCV) — clean lines, remove noise

     ↓

\[Step 2\] Symbol Detection — lightweight pre-trained model (NOT Detectron2)

     ↓

\[Step 3\] Text/Tag Extraction (OCR) — equipment tags near symbols

     ↓

\[Step 4\] Rule-based Association — link tag text to nearest symbol

     ↓

Structured Output (JSON: symbol type, tag, coordinates)

### **Step-by-Step**

**Step 1 — Pre-processing (OpenCV, local, no GPU needed)**

* Convert P\&ID pages to high-res images (300 DPI) using `pdf2image`.  
* Apply binarization (Otsu's threshold), noise removal, and line detection (Hough Transform) to isolate piping lines from symbols/text.  
* This step alone is legitimate technical work you can show in your report — most P\&ID complexity is in separating overlapping lines/text/symbols, not in the ML model itself.

**Step 2 — Symbol Detection: skip Detectron2, use one of these instead**

Pick based on your timeline:

* **Option A (fastest, recommended): YOLOv8 (Ultralytics) pre-trained \+ small fine-tune** YOLOv8 is far lighter than Detectron2 to train and has a much simpler API. You still need labeled data, but:

  * Use a small public P\&ID symbol dataset if available (search "P\&ID symbol dataset Kaggle/Roboflow" — Roboflow Universe has several open P\&ID/electrical symbol datasets you can fork).  
  * Label 100–200 symbols yourself using **Roboflow's free annotation tool** (free tier covers small projects) — much faster than raw LabelImg.  
  * Fine-tune YOLOv8n (nano variant) on **Google Colab free GPU** — a small dataset trains in under an hour on a free T4.  
  * This gives you a real trained model with real metrics (precision/recall) for your evaluation section, at a fraction of Detectron2's complexity.  
* **Option B (zero training, template matching): OpenCV Template Matching** If you genuinely don't have time to label/train anything:

  * Extract a handful of standard symbols (valve, pump, instrument bubble) as small template images from a legend/key page (P\&IDs almost always include one).  
  * Use `cv2.matchTemplate()` to find occurrences of each template across the drawing.  
  * Works well when the P\&ID follows a consistent symbol library (e.g., ISA standard shapes) and you're not dealing with rotation/scale variance.  
  * Much weaker than a trained detector, but zero training cost and defensible as a "baseline approach" in your report, with YOLO as the "improved approach" if time allows.  
* **Option C (no training, API-based): Google Cloud Vision "Object Localization"**

  * Won't know P\&ID-specific symbols (valve vs. pump) out of the box, but can detect generic shapes/text regions.  
  * Useful only as a fallback for text-region detection, not symbol classification. I'd treat this as a supplement to Option A/B, not a replacement.

**My recommendation for a final-year project:** do Option B first (gets you a working demo in days), then layer in Option A on a subset of symbols if time permits — this gives you a "baseline vs. improved" comparison, which examiners like.

**Step 3 — Text/Tag Extraction near symbols**

* Run OCR (Google Cloud Vision or PaddleOCR — see next section) on the full page.  
* OCR returns bounding boxes for each text token (e.g., "P-204").  
* Filter for tokens matching your tag-naming pattern (regex, e.g., `[A-Z]{1,2}-\d{3,4}`).

**Step 4 — Rule-based Association**

* For each detected symbol (from Step 2), find the nearest text token (Step 3\) by Euclidean distance between bounding box centers.  
* Associate them: `{symbol_type: "valve", tag: "V-204", coordinates: [...]}`.  
* This is simple, explainable geometry — no ML needed, and it's exactly the kind of practical engineering judgment call worth documenting in your report ("we chose proximity-based association over a learned association model because P\&ID layout conventions make spatial proximity a reliable heuristic").

### **Scoping note for your report**

Be explicit that production-grade P\&ID digitization (e.g., commercial tools like AVEVA's or Hexagon's P\&ID digitization products) uses much larger labeled datasets and hybrid graph-based post-processing to resolve ambiguous symbols. Framing your Option A/B pipeline as a **working proof-of-concept with a clear path to scale** is stronger than pretending it's production-ready — judges/examiners respect honest scoping.

---

## 

## **2\. Running PaddleOCR Locally as a Fallback**

The goal here isn't to replace Cloud Vision — it's to have OCR available when you're offline, testing heavily (avoiding API quota burn), or want a free-tier-independent fallback for your demo.

### **Simplified Architecture**

\[Primary Path\]  Document → Google Cloud Vision API → OCR text

\[Fallback Path\] Document → PaddleOCR (local/Colab) → OCR text

       ↓ (both converge to same downstream format)

   Unified OCR Output → Entity Extraction (Gemini)

### **Step-by-Step**

**Step 1 — Choose where PaddleOCR actually runs**

You have three realistic options, in order of ease:

* **Option A: PaddleOCR on Google Colab (free GPU/CPU)**

  * Don't install PaddleOCR on your laptop at all. Instead, run it in a Colab notebook, exposed via a small **FastAPI \+ ngrok** tunnel (or Colab's own port-forwarding), so your main backend can call it like an API during testing.  
  * Steps:  
    * In Colab: `!pip install paddlepaddle paddleocr`  
    * Write a 10-line FastAPI wrapper: accept an image, run `PaddleOCR(lang='en')`, return JSON text \+ boxes.  
    * Use `pyngrok` to expose the Colab-hosted FastAPI server as a public URL.  
    * Your main backend calls this URL when Cloud Vision quota is low or during bulk local testing.  
  * This avoids installing PaddlePaddle's (fairly heavy) dependencies on your own machine, and gives you free GPU acceleration.  
  * Caveat: Colab sessions time out (\~90 min idle, 12 hr max) — fine for development/testing, **not reliable for your live demo**. Use it as a dev-time fallback, not the demo path.  
* **Option B: PaddleOCR locally, CPU-only, on a small test subset**

If you want it truly local (no internet dependency at all):  
 pip install paddlepaddle paddleocr

*   
  * Use the lightweight model variant: `PaddleOCR(lang='en', ocr_version='PP-OCRv4', use_gpu=False)` — the "mobile"/lightweight detection+recognition models (not the server models) run acceptably on CPU for single-page documents (a few seconds per page).  
  * Realistic expectation: fine for testing 5–10 sample documents; not fine for batch-processing your whole corpus without a wait.  
  * Use this only for a small, curated "offline demo" set — e.g., 3–5 sample documents you OCR locally to show the pipeline works without any internet/API dependency, which is a nice point to make in a viva if asked "what if the API is down?"  
* **Option C: Skip local PaddleOCR entirely, treat Cloud Vision's free tier as sufficient**

  * Honestly, for a student project's document volume (tens to low hundreds of pages), Cloud Vision's free tier (1,000 units/month) is very unlikely to be exhausted during development and demo.  
  * If your only reason for wanting PaddleOCR was "avoid hitting the free tier," it may not be necessary at all. I'd only build the fallback if you specifically want to demonstrate resilience/offline capability as a feature — not just as a hedge against quota.

**Step 2 — Unify output format**

Whichever OCR source is used, normalize output into the same schema before passing to entity extraction:  
 {  "source": "cloud\_vision" | "paddleocr",  "text\_blocks": \[    {"text": "...", "bbox": \[x1,y1,x2,y2\], "confidence": 0.xx}  \]}

*   
* This means your downstream code (Gemini entity extraction, tag-symbol association) never needs to know which OCR engine ran — clean separation of concerns, and it's an easy thing to point to as "modular design" in your architecture diagram.

**Step 3 — Decide the switching logic (if you build both)**

* Simple rule-based fallback: try Cloud Vision first; if it errors (quota exceeded, network failure), catch the exception and retry via your Colab-hosted PaddleOCR endpoint.  
* This is a good, small piece of "resilience engineering" to mention in your write-up without needing complex infrastructure.

### **My recommendation for a final-year project**

Build **Option C (Cloud Vision only) as your primary, working path** — it's genuinely enough for your scale. Then, **only if you have spare time**, add Option A (Colab-hosted PaddleOCR) as a documented fallback, framed as "demonstrating provider-independence," rather than something load-bearing in your live demo. Don't burn project time on Option B (local CPU install) unless you specifically want an "offline mode" as a feature to show off.