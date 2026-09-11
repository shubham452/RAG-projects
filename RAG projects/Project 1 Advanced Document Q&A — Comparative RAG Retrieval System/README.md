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
