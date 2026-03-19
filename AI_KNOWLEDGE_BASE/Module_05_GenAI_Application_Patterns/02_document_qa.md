# Document Q&A Systems

## Architecture

```
Upload Documents → Chunk → Embed → Vector DB
                                         ↓
User Question → Embed → Search Vector DB → Top-K Chunks
                                                 ↓
                              Build Prompt (question + chunks)
                                                 ↓
                                    LLM → Answer with Citations
```

---

## Complete Document Q&A System

```python
import os
import anthropic
import chromadb
from chromadb.utils import embedding_functions
from pathlib import Path
import hashlib

client = anthropic.Anthropic()

chroma = chromadb.PersistentClient(path="./qa_db")
ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key=os.environ["OPENAI_API_KEY"],
    model_name="text-embedding-3-small"
)
collection = chroma.get_or_create_collection("documents", embedding_function=ef)


# ---- DOCUMENT LOADING ----
def load_text_file(path: str) -> str:
    with open(path, "r", encoding="utf-8") as f:
        return f.read()

def load_pdf(path: str) -> str:
    import pypdf
    reader = pypdf.PdfReader(path)
    return "\n\n".join(page.extract_text() for page in reader.pages)

def load_document(path: str) -> str:
    ext = Path(path).suffix.lower()
    if ext == ".pdf":
        return load_pdf(path)
    elif ext in (".txt", ".md"):
        return load_text_file(path)
    else:
        raise ValueError(f"Unsupported format: {ext}")


# ---- INDEXING ----
def index_document(path: str):
    text = load_document(path)
    chunks = smart_chunk(text)
    filename = Path(path).name

    # Skip if already indexed (check by hash)
    doc_hash = hashlib.md5(text.encode()).hexdigest()
    existing = collection.get(where={"doc_hash": doc_hash})
    if existing["ids"]:
        print(f"Already indexed: {filename}")
        return

    collection.add(
        documents=chunks,
        ids=[f"{filename}_{i}" for i in range(len(chunks))],
        metadatas=[{
            "source": filename,
            "chunk": i,
            "total_chunks": len(chunks),
            "doc_hash": doc_hash
        } for i in range(len(chunks))]
    )
    print(f"Indexed {len(chunks)} chunks from {filename}")


def smart_chunk(text: str, size: int = 800, overlap: int = 100) -> list[str]:
    chunks, start = [], 0
    while start < len(text):
        end = min(start + size, len(text))
        # Extend to sentence boundary
        if end < len(text):
            for sep in ['. ', '\n\n', '\n', ' ']:
                pos = text.rfind(sep, start, end)
                if pos > start + size // 2:
                    end = pos + len(sep)
                    break
        chunk = text[start:end].strip()
        if chunk:
            chunks.append(chunk)
        start = end - overlap
    return chunks


# ---- Q&A ----
def answer(question: str, top_k: int = 5) -> dict:
    # Retrieve relevant chunks
    results = collection.query(
        query_texts=[question],
        n_results=top_k,
        include=["documents", "metadatas", "distances"]
    )

    chunks = [
        {
            "text": doc,
            "source": meta["source"],
            "chunk": meta["chunk"],
            "relevance": round(1 - dist, 3)
        }
        for doc, meta, dist in zip(
            results["documents"][0],
            results["metadatas"][0],
            results["distances"][0]
        )
        if 1 - dist > 0.45  # minimum relevance threshold
    ]

    if not chunks:
        return {
            "answer": "I couldn't find relevant information in the documents to answer this question.",
            "sources": [],
            "citations": []
        }

    # Build context with source labels
    context_parts = []
    for i, chunk in enumerate(chunks):
        context_parts.append(f"[Source {i+1}: {chunk['source']}]\n{chunk['text']}")

    context = "\n\n".join(context_parts)

    # Generate answer
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system="""You are a precise document assistant. Answer questions based ONLY on the provided sources.

Rules:
- Cite sources using [Source N] notation
- If information is not in the sources, say "This information is not in the provided documents"
- Never make up or infer information not explicitly stated
- Be concise but complete""",
        messages=[{
            "role": "user",
            "content": f"Sources:\n{context}\n\nQuestion: {question}"
        }]
    )

    return {
        "answer": response.content[0].text,
        "sources": list(set(c["source"] for c in chunks)),
        "citations": [{"source": c["source"], "relevance": c["relevance"]} for c in chunks]
    }


# ---- USAGE ----
# Index documents
# index_document("employee_handbook.pdf")
# index_document("product_manual.txt")

# Ask questions
result = answer("What is the vacation policy?")
print(result["answer"])
print(f"\nSources: {result['sources']}")
```

---

## Listing Indexed Documents

```python
def list_documents() -> list[str]:
    all_docs = collection.get()
    sources = set(m["source"] for m in all_docs["metadatas"])
    return sorted(sources)

def remove_document(filename: str):
    all_docs = collection.get(where={"source": filename})
    if all_docs["ids"]:
        collection.delete(ids=all_docs["ids"])
        print(f"Removed {len(all_docs['ids'])} chunks from {filename}")
```

---

## Document Q&A API (FastAPI)

```python
from fastapi import FastAPI, UploadFile, File
import tempfile

app = FastAPI()

@app.post("/upload")
async def upload_document(file: UploadFile = File(...)):
    with tempfile.NamedTemporaryFile(delete=False, suffix=Path(file.filename).suffix) as tmp:
        content = await file.read()
        tmp.write(content)
        tmp_path = tmp.name

    index_document(tmp_path)
    return {"message": f"Indexed {file.filename}"}

@app.get("/ask")
async def ask_question(q: str):
    return answer(q)

@app.get("/documents")
async def get_documents():
    return {"documents": list_documents()}
```
