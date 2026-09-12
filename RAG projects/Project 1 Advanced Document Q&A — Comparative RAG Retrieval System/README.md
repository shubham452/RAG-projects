

```text
Same documents
      │
      ▼
 Same question
      │
      ├───────────────┬──────────────────┬────────────────────────┐
      ▼               ▼                  ▼                        ▼
   NAIVE           HYBRID        HYBRID + RERANKER        ADVANCED RAG
      │               │                  │                        │
 Vector search   Vector + BM25    Vector + BM25            Vector + BM25
      │               │                  │                    + RRF
    Top 5             RRF                RRF                  + Reranking
      │             Top 20             Top 20                 + Compression
      │               │               Reranker                       │
      │               │                Top 5                         Top 5
      │               │                  │                            │
      └───────────────┴──────────────────┴────────────────────────────┘
                                      │
                                      ▼
                             LLM generates answer
                                      │
                                      ▼
                              RAGAS Evaluation
```

### What we're comparing

| Pipeline              | Purpose                                                       |
| --------------------- | ------------------------------------------------------------- |
| **Naive RAG**         | Baseline: how good is simple vector retrieval?                |
| **Hybrid RAG**        | Does vector + BM25 improve retrieval?                         |
| **Hybrid + Reranker** | Does cross-encoder ranking improve it further?                |
| **Advanced RAG**      | Does contextual compression improve the final context/answer? |



```text
Question
   │
   ├── Naive ──────────► Answer A
   ├── Hybrid ─────────► Answer B
   ├── Reranked ───────► Answer C
   └── Advanced ───────► Answer D
                         │
                         ▼
                    Compare with RAGAS
```

 we do:

```text
Same question → four independent pipelines → compare results
```

That gives you a proper experimental progression:

**Naive → Hybrid → Reranking → Advanced**
Yes. For the **backend**, I would divide Project 1 into these phases, in this exact order.

# Project 1 Backend Roadmap

**Advanced Document Q&A — Comparative RAG Retrieval System**

```text
PHASE 0
Development Setup
      ↓
PHASE 1
FastAPI Foundation
      ↓
PHASE 2
Database & Data Models
      ↓
PHASE 3
Document Ingestion
      ↓
PHASE 4
Embeddings + Vector DB
      ↓
PHASE 5
Naive RAG
      ↓
PHASE 6
Hybrid RAG
      ↓
PHASE 7
Re-ranking
      ↓
PHASE 8
Contextual Compression
      ↓
PHASE 9
RAG Pipelines / Mode Selection
      ↓
PHASE 10
Evaluation + RAGAS
      ↓
PHASE 11
Production Improvements
      ↓
PHASE 12
Deployment
```

---

# Phase 0 — Development Setup

This is the setup you're doing now.

```text
Python
Virtual Environment
Git
VS Code
requirements.txt
.env
.gitignore
Docker
Docker Compose
```

Infrastructure:

```text
Docker
 ├── PostgreSQL
 └── Qdrant
```

### Status

You have already started this.

---

# Phase 1 — FastAPI Foundation

This is the **standard FastAPI architecture** we discussed.

```text
app/
│
├── main.py
│
├── api/
├── core/
├── db/
├── models/
├── schemas/
├── services/
└── repositories/
```

Implement:

```text
FastAPI application
CORS
Configuration
Logging
Health checks
Database connection
Qdrant connection
API versioning
```

Endpoints initially:

```http
GET /api/v1/health
GET /api/v1/health/qdrant
```

### Goal

Get:

```text
FastAPI
   │
   ├── PostgreSQL ✓
   └── Qdrant ✓
```

working.

**You are here.**

---

# Phase 2 — Database & Data Models

Now design the application's persistent data.

Initially:

```text
Document
DocumentChunk
Query
Answer
```

Potential structure:

```text
documents
---------
id
filename
file_path
file_type
file_size
status
created_at
updated_at
```

```text
document_chunks
---------------
id
document_id
chunk_index
text
page_number
section
token_count
created_at
```

Later:

```text
queries
answers
evaluation_results
```

Use:

```text
SQLAlchemy
+
Alembic
+
PostgreSQL
```

Flow:

```text
FastAPI
   ↓
SQLAlchemy
   ↓
PostgreSQL
```

And migrations:

```text
Model change
    ↓
Alembic migration
    ↓
PostgreSQL
```

---

# Phase 3 — Document Ingestion

Now the system actually accepts documents.

Endpoint:

```http
POST /api/v1/documents/upload
```

Flow:

```text
PDF
 │
 ▼
Upload API
 │
 ▼
Validate file
 │
 ▼
Save document
 │
 ▼
Parse document
 │
 ▼
Extract text
 │
 ▼
Create chunks
 │
 ▼
Store chunks
```

Start with:

```text
PDF
```

Then later:

```text
DOCX
TXT
PPTX
```

Document status should be something like:

```text
uploaded
processing
processed
failed
```

---

# Phase 4 — Embeddings + Vector Database

Now we introduce the actual **vector component**.

Flow:

```text
Document
   ↓
Chunks
   ↓
Embedding Model
   ↓
Vectors
   ↓
Qdrant
```

For example:

```text
Chunk:

"Employees are entitled to 20 days of annual leave."

             ↓ embedding

[0.021, -0.381, 0.722, ...]
```

Qdrant stores:

```text
Vector
+
chunk_id
+
document_id
+
page_number
+
metadata
```

Create a proper service:

```python
EmbeddingService
```

and:

```python
VectorStoreService
```

---

# Phase 5 — Naive RAG

**This is the first actual RAG pipeline.**

Flow:

```text
User Question
      ↓
Embedding
      ↓
Qdrant
      ↓
Similarity Search
      ↓
Top 5 chunks
      ↓
Prompt
      ↓
LLM
      ↓
Answer
```

Create:

```text
retrieval/
    vector.py

rag/
    naive.py

generation/
    llm.py
    prompts.py
```

Endpoint:

```http
POST /api/v1/query
```

Request:

```json
{
  "question": "What is the leave policy?",
  "document_ids": ["doc_123"],
  "mode": "naive"
}
```

Response:

```json
{
  "answer": "...",
  "sources": [
    {
      "document_id": "doc_123",
      "page": 14,
      "chunk_id": "chunk_45"
    }
  ]
}
```

At this point:

> **You have a complete working Naive RAG application.**

---

# Phase 6 — Hybrid RAG

Now we add **BM25**.

The query goes to two retrieval systems:

```text
                 Query
                /     \
               ▼       ▼
          Vector       BM25
          Search       Search
               \       /
                ▼     ▼
                   RRF
                    ↓
                 Top 20
```

Implement:

```text
retrieval/
├── vector.py
├── bm25.py
├── hybrid.py
└── rrf.py
```

### Important

BM25 needs an indexed representation of your chunks.

For the first version, you can build an in-memory BM25 index from PostgreSQL chunks.

Later, if you want production-scale lexical retrieval, move it to a dedicated search engine.

---

# Phase 7 — Re-ranking

Now improve the Hybrid pipeline.

```text
Vector + BM25
      ↓
     RRF
      ↓
  20 chunks
      ↓
Cross Encoder
      ↓
   Top 5
```

Create:

```text
reranking/
    cross_encoder.py
```

The cross-encoder evaluates:

```text
(query, chunk)
```

rather than independently embedding them.

Your pipeline becomes:

```text
Hybrid
  ↓
RRF
  ↓
20
  ↓
Reranker
  ↓
5
```

---

# Phase 8 — Contextual Compression

Now reduce irrelevant context.

```text
Top 5 chunks
      ↓
Compression LLM
      ↓
Relevant sentences
      ↓
Smaller context
      ↓
Final LLM
```

Create:

```text
compression/
    contextual_compressor.py
```

For example:

```text
Original chunk = 800 tokens

        ↓ compression

Relevant content = 150 tokens
```

This gives you your final:

> **Advanced RAG**

pipeline.

---

# Phase 9 — RAG Pipeline Architecture

Now is the time to make the different modes cleanly selectable.

I recommend:

```text
rag/
├── base.py
├── naive.py
├── hybrid.py
├── reranked.py
└── advanced.py
```

Conceptually:

```python
class RAGPipeline:

    async def run(self, query):
        ...
```

Then:

```text
NaivePipeline
HybridPipeline
RerankedPipeline
AdvancedPipeline
```

Your API receives:

```json
{
  "question": "...",
  "mode": "advanced"
}
```

and the backend selects:

```text
naive
   → NaivePipeline

hybrid
   → HybridPipeline

reranked
   → RerankedPipeline

advanced
   → AdvancedPipeline
```

This is the point where the architecture becomes clean enough for your React frontend to control.

---

# Phase 10 — Evaluation + RAGAS

Now we prove whether the improvements actually work.

Create:

```text
evaluation/
├── dataset.py
├── runner.py
├── metrics.py
└── ragas.py
```

Dataset:

```text
30–50 questions
```

Each question might contain:

```json
{
  "question": "How many remote work days are allowed?",
  "ground_truth": "Three days per week",
  "document_id": "employee_handbook"
}
```

Run:

```text
Question
   │
   ├── Naive
   ├── Hybrid
   ├── Reranked
   └── Advanced
```

Then measure things such as:

```text
Faithfulness
Answer Relevancy
Context Precision
Context Recall
```

This produces the data for your final comparison.

---

# Phase 11 — Production Improvements

Only after the core RAG works.

Add:

### Background processing

Instead of:

```text
Upload
 ↓
Process everything
 ↓
Response
```

use:

```text
Upload
 ↓
Create job
 ↓
Background worker
 ↓
Parse
 ↓
Chunk
 ↓
Embed
 ↓
Qdrant
```

Possible stack:

```text
FastAPI
   ↓
Redis
   ↓
Celery / worker
```

---

### Caching

Cache things such as:

```text
Embeddings
Repeated queries
Document processing
```

---

### Authentication

If the application becomes multi-user:

```text
User
 ↓
JWT
 ↓
FastAPI
 ↓
Documents belonging to user
```

---

### Observability

Track:

```text
query
retrieval latency
number of chunks
reranking latency
LLM latency
token usage
cost
pipeline mode
answer
sources
```

This will be very useful when comparing pipelines.

---

# Phase 12 — Deployment

Finally:

```text
                 Internet
                    │
                    ▼
             React Frontend
                    │
                    ▼
              FastAPI API
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
   PostgreSQL     Qdrant       Redis
                                │
                                ▼
                              Worker
```

Then you can demonstrate your three deployment concepts:

```text
Cloud
Hybrid
Local
```

---

# Complete Backend Roadmap

So I would keep this as your master checklist:

```text
┌─────────────────────────────────────┐
│  PROJECT 1 BACKEND                  │
└─────────────────────────────────────┘

01. Development Setup                 ✅
    ├── Python
    ├── venv
    ├── Git
    ├── .env
    ├── Docker
    └── Docker Compose

02. FastAPI Foundation                🔄
    ├── FastAPI
    ├── Config
    ├── CORS
    ├── Logging
    ├── PostgreSQL connection
    └── Qdrant connection

03. Database Layer                    ⬜
    ├── Models
    ├── SQLAlchemy
    ├── Alembic
    └── PostgreSQL schema

04. Document Ingestion                ⬜
    ├── Upload
    ├── Validation
    ├── PDF parsing
    ├── Text extraction
    ├── Chunking
    └── Metadata

05. Embedding + Vector Store          ⬜
    ├── Embedding service
    ├── Qdrant collection
    ├── Vector insertion
    └── Vector search

06. NAIVE RAG                         ⬜
    ├── Query embedding
    ├── Vector retrieval
    ├── Top-K
    ├── Prompt
    └── LLM answer

07. HYBRID RAG                        ⬜
    ├── BM25
    ├── Vector search
    ├── RRF
    └── Top-K

08. RE-RANKING                        ⬜
    ├── Cross encoder
    ├── Candidate scoring
    └── Top-K selection

09. CONTEXTUAL COMPRESSION            ⬜
    ├── Relevant sentence extraction
    └── Reduced context

10. ADVANCED RAG                      ⬜
    └── Full pipeline

11. PIPELINE ORCHESTRATION            ⬜
    ├── Naive
    ├── Hybrid
    ├── Reranked
    └── Advanced

12. EVALUATION                        ⬜
    ├── Dataset
    ├── Run pipelines
    ├── RAGAS
    ├── Metrics
    └── Comparison

13. PRODUCTION                        ⬜
    ├── Redis
    ├── Background workers
    ├── Caching
    ├── Authentication
    ├── Rate limiting
    └── Observability

14. DEPLOYMENT                        ⬜
    ├── Docker
    ├── Cloud
    ├── Hybrid
    └── Local
```

### The key learning order

Don't jump directly to Advanced RAG.

**Build → understand → test → improve → compare.**

```text
FastAPI Foundation
        ↓
Database
        ↓
Ingestion
        ↓
Vectors
        ↓
Naive RAG          ← baseline
        ↓
Hybrid RAG         ← Vector + BM25
        ↓
Re-ranking         ← better ordering
        ↓
Compression        ← better context
        ↓
Advanced RAG       ← complete pipeline
        ↓
RAGAS              ← prove improvement
```

This order will let you understand **every layer instead of hiding the important RAG concepts behind LangChain/LlamaIndex abstractions**.


