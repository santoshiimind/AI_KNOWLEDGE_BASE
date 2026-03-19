# Module 03 — RAG (Retrieval-Augmented Generation)

RAG solves the problem of LLMs not knowing about your private data or recent events.
Instead of fine-tuning, you retrieve relevant documents at query time and inject them into the prompt.

---

## Topics

| # | Topic | Status |
|---|-------|--------|
| 1 | [Vector Databases](./01_vector_databases.md) | ⬜ |
| 2 | [Embeddings & Semantic Search](./02_embeddings_semantic_search.md) | ⬜ |
| 3 | [Chunking Strategies](./03_chunking_strategies.md) | ⬜ |
| 4 | [Building a RAG Pipeline](./04_rag_pipeline.md) | ⬜ |

---

## Learning Goals

- Understand why RAG exists and when to use it vs fine-tuning
- Create and store embeddings in a vector database
- Retrieve semantically relevant documents for a query
- Build an end-to-end Q&A system over your own documents
- Know common failure modes and how to improve retrieval quality

---

## RAG in One Diagram

```
User Query
    ↓
[Embed Query] → Query Vector
    ↓
[Vector DB Similarity Search] → Top-K Relevant Chunks
    ↓
[Build Prompt: Query + Chunks]
    ↓
[LLM] → Answer grounded in your documents
```

---

## Hands-On Practice

| Type | File | Description |
|------|------|-------------|
| Exercises | [exercises.md](./exercises.md) | 5 focused coding challenges |
| Project | [project.md](./project.md) | One substantial mini-project |

Complete all topic files before attempting the exercises.
Finish the exercises before starting the project.
