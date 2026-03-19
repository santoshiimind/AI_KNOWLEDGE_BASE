# Local Models with Ollama

## Why Run Models Locally?

- **Privacy**: data never leaves your machine
- **Cost**: no per-token fees after setup
- **Offline**: works without internet
- **Customization**: run fine-tuned models
- **Latency**: no network round-trip

**Trade-offs**: slower than cloud APIs, requires decent hardware, smaller context windows.

---

## Ollama Setup

```bash
# Install Ollama (Windows/Mac/Linux)
# Download from: https://ollama.ai

# Pull a model
ollama pull llama3.2          # 3B params, fast, good for simple tasks
ollama pull llama3.1:8b       # 8B params, balanced
ollama pull llama3.1:70b      # 70B params, best quality (needs ~40GB RAM)
ollama pull mistral           # 7B, fast and capable
ollama pull phi3.5            # 3.8B Microsoft model, excellent for its size
ollama pull qwen2.5-coder     # specialized for code
ollama pull nomic-embed-text  # for embeddings

# List installed models
ollama list

# Run interactively
ollama run llama3.2
```

---

## Using Ollama with Python

### Direct API

```python
import requests
import json

def ollama_chat(prompt: str, model: str = "llama3.2") -> str:
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": model,
            "prompt": prompt,
            "stream": False
        }
    )
    return response.json()["response"]

# Chat format
def ollama_messages(messages: list, model: str = "llama3.2") -> str:
    response = requests.post(
        "http://localhost:11434/api/chat",
        json={
            "model": model,
            "messages": messages,
            "stream": False
        }
    )
    return response.json()["message"]["content"]
```

### OpenAI-Compatible API

Ollama exposes an OpenAI-compatible endpoint — use the OpenAI SDK:

```python
from openai import OpenAI

# Point to local Ollama
client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # required but ignored
)

response = client.chat.completions.create(
    model="llama3.2",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain recursion."}
    ]
)

print(response.choices[0].message.content)
```

### Streaming

```python
client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

stream = client.chat.completions.create(
    model="llama3.2",
    messages=[{"role": "user", "content": "Write a poem about coding."}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

---

## Local Embeddings

```python
import requests
import numpy as np

def local_embed(text: str, model: str = "nomic-embed-text") -> list[float]:
    response = requests.post(
        "http://localhost:11434/api/embeddings",
        json={"model": model, "prompt": text}
    )
    return response.json()["embedding"]

# Use in a RAG pipeline — no OpenAI embeddings needed
embedding = local_embed("Hello, world!")
print(f"Dimensions: {len(embedding)}")
```

---

## LangChain with Ollama

```python
from langchain_community.llms import Ollama
from langchain_community.embeddings import OllamaEmbeddings

llm = Ollama(model="llama3.1:8b", temperature=0)
embeddings = OllamaEmbeddings(model="nomic-embed-text")

# Use exactly like cloud models
response = llm.invoke("What is machine learning?")
embedding = embeddings.embed_query("Hello world")
```

---

## Model Selection Guide

| Model | Size | RAM Needed | Best For |
|-------|------|-----------|---------|
| phi3.5 | 3.8B | 4GB | Fast tasks, low resource |
| llama3.2:3b | 3B | 3GB | Quick responses |
| mistral:7b | 7B | 8GB | General use |
| llama3.1:8b | 8B | 10GB | Balanced quality/speed |
| llama3.1:70b | 70B | 48GB | Best quality |
| qwen2.5-coder | 7B | 8GB | Code generation |
| nomic-embed-text | - | 1GB | Embeddings |

---

## Fully Local RAG Pipeline

```python
from openai import OpenAI
import chromadb
from chromadb.utils import embedding_functions

# Everything local — no internet required
client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

# Custom embedding function using Ollama
class OllamaEmbeddingFunction(embedding_functions.EmbeddingFunction):
    def __call__(self, texts):
        import requests
        embeddings = []
        for text in texts:
            resp = requests.post(
                "http://localhost:11434/api/embeddings",
                json={"model": "nomic-embed-text", "prompt": text}
            )
            embeddings.append(resp.json()["embedding"])
        return embeddings

chroma = chromadb.PersistentClient(path="./local_rag")
collection = chroma.get_or_create_collection(
    "docs",
    embedding_function=OllamaEmbeddingFunction()
)

def local_rag_answer(question: str) -> str:
    results = collection.query(query_texts=[question], n_results=3)
    context = "\n\n".join(results["documents"][0])

    response = client.chat.completions.create(
        model="llama3.1:8b",
        messages=[
            {"role": "system", "content": f"Answer using only this context:\n{context}"},
            {"role": "user", "content": question}
        ]
    )
    return response.choices[0].message.content
```

---

## Hardware Recommendations

| Setup | Recommended Model |
|-------|-----------------|
| 8GB RAM laptop | phi3.5, llama3.2:3b |
| 16GB RAM laptop | mistral:7b, llama3.1:8b |
| 32GB RAM desktop | llama3.1:13b |
| GPU (8GB VRAM) | llama3.1:8b (much faster) |
| GPU (24GB VRAM) | llama3.1:70b quantized |
