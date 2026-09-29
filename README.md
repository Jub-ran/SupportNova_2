# SupportNova

Generative AI Customer Complaint Resolution Intelligence

## Project Overview
SupportNova is a Generative AI-powered customer complaint management system for NovaMart (a fictional e-commerce organization). It processes customer complaints, retrieves relevant company policies via RAG, generates professional customer responses, and features an **independent Python Ground-Truth Validation Pipeline** to ensure GenAI output accuracy against strict business rules.

## Architecture
The application features two pipelines:
1. **GenAI Complaint Intelligence Pipeline**: Extracts issues, sentiment, urgency, entities, and suggests resolutions.
2. **Python Ground-Truth Validation Pipeline**: Validates GenAI suggestions against deterministic logic, policies, and a rule matrix to prevent hallucinations, contradictions, or unsupported promises.

## Features
- Upload and chunk company knowledge base documents (PDF/DOCX).
- Process and sanitize customer complaints.
- RAG for retrieving relevant policies.
- Structured JSON output via Google Gemini API (or other providers).
- Ground truth validation (no AI in the loop).
- Mismatch detection, prompt injection defense, hallucination detection.
- Department routing & escalation rules.
- Administrator and agent dashboards.

## Technology Stack
- Backend: Python 3.x, FastAPI
- Frontend: HTML5, CSS3, Bootstrap, JavaScript
- Database: SQLite (ready for PostgreSQL)
- GenAI: Google Gemini API (configurable via `.env`)
- Retrieval: FAISS-cpu

## Installation & Setup

1. Create a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate  # on Windows: venv\Scripts\activate
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Environment Variables:
   Copy `.env.example` to `.env` and fill in the values.
   ```bash
   cp .env.example .env
   ```

4. Database Setup:
   ```bash
   # Run initial migrations (To be implemented)
   ```

5. Run Application locally:
   ```bash
   uvicorn app.main:app --reload
   ```

## Creating Admin Account
(To be implemented)

## Running Tests
   ```bash
   pytest
   ```

## Phase 4: RAG Implementation Details

### What RAG is in SupportNova
RAG (Retrieval-Augmented Generation) in SupportNova acts as the strict intelligence layer for factual company knowledge. It maps user queries to semantic vectors, retrieving the `Top-K` matching `ACTIVE` policy chunks to pass as unalterable *context* to the Phase 5 GenAI Engine.

### Architecture
- **Embedding Model:** Google GenAI `text-embedding-004` (Isolated behind `EmbeddingProvider` for fallback/offline swapping).
- **Vector Database:** `FAISS-cpu` operating entirely locally without remote storage. Uses Inner Product cosine similarity.
- **Indexing Process:** Indexes ONLY chunks of documents present in the active Knowledge Base database. Vector IDs identically map to relational SQLite `DocumentChunk.id`. 
- **Retrieval Process:** Converts query via `get_embedding`, searches FAISS for raw matches, queries relational DB for metadata mapping, applies `RAG_MIN_SIMILARITY`, and isolates items matching filter requirements. 
- **Version Filtering:** The service explicitly drops `DRAFT` or `OUTDATED` document chunks from standard retrieval payloads, returning only `ACTIVE` rules to avoid contradictory GenAI outputs.
- **Source Traceability:** Every chunk maintains a strict lineage to Document Name, Version ID, Section, Page Number, and Effective Date, ensuring accurate citations.
- **Prompt-Injection Defense:** Retrieved textual data is explicitly designated as `UNTRUSTED DATA` in the context builder. Malicious documents advising the system to bypass rules will be processed safely as data and isolated from system directives.

### API Endpoints
- `POST /rag/search`: Accepts `{"query": "...", "top_k": 5}` and returns retrieved knowledge blocks.
- `POST /rag/rebuild`: Protected admin endpoint reconstructing the `.bin` FAISS index directly from the persistent SQLite schema.

### How to Rebuild the Index
Using the API: Send a `POST` request to `/rag/rebuild` (requires `Administrator` token), or click the `Rebuild Index` button in the `/rag` Administrator interface.

### How to Test RAG
Navigate to `/rag` in the frontend (Admin testing UI). You can enter natural language queries like `"Returns limit"` and see structurally retrieved vectors, complete with scores and version source data.
