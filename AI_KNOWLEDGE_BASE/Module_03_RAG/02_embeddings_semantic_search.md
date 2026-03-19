# Embeddings & Semantic Search

## What Are Embeddings?

Embeddings convert text into a fixed-size vector of numbers that **captures meaning**.
Similar sentences end up close together in vector space.

```
"The cat sat on the mat"     → [0.12, -0.45, 0.87, ...]
"A feline rested on the rug" → [0.11, -0.43, 0.85, ...]  ← very similar!
"Stock markets rose today"   → [-0.67, 0.23, -0.12, ...]  ← very different
```

---

## Embedding Models

### OpenAI Embeddings

```python
from openai import OpenAI

client = OpenAI()

def embed(text: str) -> list[float]:
    response = client.embeddings.create(
        model="text-embedding-3-small",  # 1536 dims, cheap
        # model="text-embedding-3-large",  # 3072 dims, better quality
        input=text
    )
    return response.data[0].embedding

vector = embed("Hello, world!")
print(f"Dimensions: {len(vector)}")  # 1536
```

### Anthropic (via Voyage AI)
Anthropic recommends Voyage AI for embeddings:

```bash
pip install voyageai
```

```python
import voyageai

client = voyageai.Client(api_key="your-voyage-key")

result = client.embed(
    ["Hello world", "How are you?"],
    model="voyage-3",
    input_type="document"
)

print(result.embeddings[0])  # vector for first text
```

### Open Source (Local) — Sentence Transformers

```bash
pip install sentence-transformers
```

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")  # fast, 384 dims
# model = SentenceTransformer("all-mpnet-base-v2")  # better quality, 768 dims

sentences = ["This is an example sentence", "Each sentence is converted"]
embeddings = model.encode(sentences)
print(embeddings.shape)  # (2, 384)
```

---

## Semantic Search from Scratch

```python
import numpy as np
from openai import OpenAI

client = OpenAI()

def cosine_similarity(a: list, b: list) -> float:
    a, b = np.array(a), np.array(b)
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

def embed(text: str) -> list[float]:
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding

# Your document corpus
documents = [
    "Python is a high-level programming language.",
    "Machine learning uses algorithms to learn from data.",
    "Paris is the capital city of France.",
    "Transformers are a type of neural network architecture.",
    "The Eiffel Tower is located in Paris."
]

# Embed all documents (do this once, then store in vector DB)
doc_embeddings = [(doc, embed(doc)) for doc in documents]

# Semantic search
def search(query: str, top_k: int = 3):
    query_embedding = embed(query)
    scores = [
        (doc, cosine_similarity(query_embedding, emb))
        for doc, emb in doc_embeddings
    ]
    scores.sort(key=lambda x: x[1], reverse=True)
    return scores[:top_k]

results = search("What neural network is used in AI?")
for doc, score in results:
    print(f"{score:.3f} | {doc}")
```

---

## Embedding Best Practices

### 1. Embed Documents vs Queries Differently
Some models have separate modes for documents and search queries:

```python
# When indexing documents
result = client.embed(texts, model="voyage-3", input_type="document")

# When searching
result = client.embed([query], model="voyage-3", input_type="query")
```

### 2. Normalize Embeddings
Many vector DBs work better with normalized vectors:

```python
import numpy as np

def normalize(vector: list) -> list:
    arr = np.array(vector)
    return (arr / np.linalg.norm(arr)).tolist()
```

### 3. Batch Your Embedding Requests
Don't embed one document at a time:

```python
# Inefficient: one API call per document
embeddings = [embed(doc) for doc in documents]

# Efficient: one API call for all documents
response = client.embeddings.create(
    model="text-embedding-3-small",
    input=documents  # pass list of strings
)
embeddings = [item.embedding for item in response.data]
```

### 4. Cache Embeddings
Embeddings are deterministic — cache them to avoid re-computing:

```python
import hashlib
import json

embedding_cache = {}

def embed_cached(text: str) -> list[float]:
    key = hashlib.md5(text.encode()).hexdigest()
    if key not in embedding_cache:
        embedding_cache[key] = embed(text)
    return embedding_cache[key]
```

---

## Embedding Model Comparison

| Model | Dimensions | Speed | Quality | Cost |
|-------|-----------|-------|---------|------|
| text-embedding-3-small | 1536 | Fast | Good | Low |
| text-embedding-3-large | 3072 | Medium | Best OpenAI | Medium |
| voyage-3 | 1024 | Fast | Excellent | Low |
| all-MiniLM-L6-v2 | 384 | Very Fast | Good | Free (local) |
| all-mpnet-base-v2 | 768 | Fast | Better | Free (local) |

---

## Hybrid Search (Vector + Keyword)

Combine semantic search with traditional keyword (BM25) search for better results:

```python
# Semantic score (what it means)
semantic_score = cosine_similarity(query_vec, doc_vec)

# Keyword score (exact word match - BM25)
from rank_bm25 import BM25Okapi
bm25 = BM25Okapi([doc.split() for doc in documents])
keyword_scores = bm25.get_scores(query.split())

# Combine (Reciprocal Rank Fusion or weighted average)
final_score = 0.7 * semantic_score + 0.3 * keyword_score
```
