# Module 03 — Project: Personal Knowledge Base Q&A

## What You'll Build

A CLI tool that lets you index your own documents (PDFs, text files, Markdown) and ask questions about them. Think of it as a private, local ChatGPT that only knows what you feed it.

---

## Features to Implement

### Core (Required)
- [ ] Index any `.txt`, `.md`, or `.pdf` file from a folder
- [ ] Persist the vector store to disk (so you don't re-index every run)
- [ ] Answer questions with source citations (filename + chunk number)
- [ ] Skip already-indexed files (track by content hash)
- [ ] CLI commands: `index <folder>`, `ask <question>`, `list` (show indexed docs), `remove <file>`

### Stretch Goals
- [ ] Relevance threshold — refuse to answer if no chunk scores above 0.5
- [ ] `--top-k` flag to control how many chunks to retrieve
- [ ] Reranking: after retrieving top-10, use a cross-encoder to rerank and keep top-3
- [ ] Answer confidence score displayed to user
- [ ] Support URLs: `index https://example.com/article`

---

## Project Structure

```
knowledge_base/
├── main.py           # CLI entry point
├── indexer.py        # document loading + chunking + embedding + storing
├── retriever.py      # semantic search
├── qa.py             # prompt building + LLM call
├── store.py          # ChromaDB wrapper
└── data/
    └── chroma_db/    # persisted vector store
```

---

## CLI Design

```bash
# Index all documents in a folder
$ python main.py index ./my_docs

Indexing employee_handbook.pdf... 47 chunks added
Indexing meeting_notes.txt... 12 chunks added
Indexing product_roadmap.md... 23 chunks added
Done. Total: 82 chunks across 3 documents.

# Ask a question
$ python main.py ask "What is our work from home policy?"

Answer: Employees may work from home up to 3 days per week with manager approval...

Sources:
  - employee_handbook.pdf (chunks 12, 13)

# List indexed documents
$ python main.py list

Indexed documents:
  employee_handbook.pdf    47 chunks
  meeting_notes.txt        12 chunks
  product_roadmap.md       23 chunks

# Remove a document
$ python main.py remove meeting_notes.txt
Removed 12 chunks.
```

---

## Key Implementation Notes

```python
# indexer.py — track indexed files to avoid duplicates
import hashlib, json, os

INDEX_REGISTRY = "data/indexed_files.json"

def get_file_hash(path: str) -> str:
    return hashlib.md5(open(path, "rb").read()).hexdigest()

def is_already_indexed(path: str) -> bool:
    registry = load_registry()
    return get_file_hash(path) in registry.values()

def register_file(path: str):
    registry = load_registry()
    registry[os.path.basename(path)] = get_file_hash(path)
    save_registry(registry)
```

---

## What You'll Learn

- Full RAG pipeline from scratch
- Persistent vector storage
- Chunking real-world documents
- Source attribution in LLM answers
- Building a useful developer tool
