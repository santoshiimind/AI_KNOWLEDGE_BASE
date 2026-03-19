# Module 07 — Project: Private Local AI Assistant

## What You'll Build

A fully private, offline-capable AI assistant that runs entirely on your machine. No data leaves your computer. Uses local models via Ollama, local embeddings, and local vector storage.

This is the capstone project combining everything from Module 01-07 — but with 100% local infrastructure.

---

## Features to Implement

### Core (Required)
- [ ] Chat with a local LLM (llama3.1:8b or phi3.5 via Ollama)
- [ ] RAG over local documents (local embeddings + ChromaDB)
- [ ] MCP server that Claude Desktop can connect to
- [ ] Streaming responses in terminal
- [ ] Conversation history saved to local SQLite
- [ ] Works fully offline

### Stretch Goals
- [ ] Swap models on the fly: `/model phi3.5` vs `/model llama3.1:8b`
- [ ] Voice input via Whisper (local `whisper.cpp`)
- [ ] MCP tool: `run_python_code(code)` — execute code in a sandbox
- [ ] Web UI using only vanilla HTML/JS (no framework, no CDN)
- [ ] Automatically index a watched folder (re-index when files change)
- [ ] Export full chat history as Markdown

---

## Architecture

```
User (Terminal or Browser)
        ↓
  Local FastAPI Server (localhost:8000)
        ├── Chat Handler
        │     ├── Semantic Cache (ChromaDB)
        │     ├── RAG Retriever (ChromaDB + local embeddings)
        │     └── Ollama LLM (localhost:11434)
        ├── Document Indexer
        │     ├── File Watcher (watchdog)
        │     └── Chunker + nomic-embed-text
        └── History DB (SQLite)
```

---

## Project Structure

```
local_assistant/
├── main.py               # FastAPI app + CLI entry point
├── config.py             # model names, paths, settings
├── llm.py                # Ollama wrapper (chat + stream)
├── embedder.py           # local embeddings via Ollama
├── rag/
│   ├── indexer.py        # document loading + chunking + storing
│   └── retriever.py      # similarity search
├── memory/
│   ├── history.db        # SQLite conversation history
│   └── history.py        # history CRUD
├── cache/
│   └── semantic_cache.py # local semantic cache
├── mcp_server/
│   └── server.py         # MCP server for Claude Desktop
├── frontend/
│   └── index.html        # optional web UI
└── docs/                 # watched folder for auto-indexing
```

---

## Config

```python
# config.py
CHAT_MODEL = "llama3.1:8b"          # change to phi3.5 for faster/lighter
EMBED_MODEL = "nomic-embed-text"    # local embeddings, no API needed
OLLAMA_BASE_URL = "http://localhost:11434"
DOCS_FOLDER = "./docs"
DB_PATH = "./memory/history.db"
CHROMA_PATH = "./rag/chroma_db"
CACHE_SIMILARITY_THRESHOLD = 0.93
```

---

## Core LLM Wrapper

```python
# llm.py
import requests
from config import CHAT_MODEL, OLLAMA_BASE_URL

def chat_stream(messages: list, system: str = None):
    """Yield tokens from Ollama streaming response."""
    if system:
        messages = [{"role": "system", "content": system}] + messages

    response = requests.post(
        f"{OLLAMA_BASE_URL}/api/chat",
        json={"model": CHAT_MODEL, "messages": messages, "stream": True},
        stream=True
    )

    for line in response.iter_lines():
        if line:
            import json
            data = json.loads(line)
            if "message" in data:
                yield data["message"]["content"]
            if data.get("done"):
                break
```

---

## Sample Session

```bash
$ python main.py

=== Local AI Assistant ===
Model: llama3.1:8b (local) | Docs: 3 indexed | Offline: Yes

You: what does our employee handbook say about sick leave?

[Searching local docs...]
[Found 2 relevant chunks from: employee_handbook.pdf]