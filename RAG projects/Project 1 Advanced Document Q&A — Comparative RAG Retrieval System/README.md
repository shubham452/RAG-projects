

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


