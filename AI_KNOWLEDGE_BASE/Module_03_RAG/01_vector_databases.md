# Vector Databases

## What Is a Vector Database?

A vector database stores and searches **high-dimensional vectors** (embeddings) efficiently.
Unlike traditional databases that search for exact matches, vector DBs search by **similarity**.

---

## How Similarity Search Works

1. Text → Embedding model → Vector (e.g., [0.12, -0.45, 0.33, ...] with 1536 dimensions)
2. Store vectors with metadata (source, page, date, etc.)
3. At query time: embed the query → find nearest vectors → return their associated text

**Distance metrics:**
- **Cosine similarity**: angle between vectors (most common for text)
- **Euclidean distance**: straight-line distance
- **Dot product**: similar to cosine but magnitude-sensitive

---

## Major Vector Databases

### Chroma (Best for getting started)
- Open source, runs locally, no setup required
- Great for development and small-to-medium projects

```bash
pip install chromadb
```

```python
import chromadb
from chromadb.utils import embedding_functions

# Create in-memory client (or persistent with path)
client = chromadb.Client()
# client = chromadb.PersistentClient(path="./chroma_db")

# Use OpenAI embeddings
openai_ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key="your-key",
    model_name="text-embedding-3-small"
)

collection = client.create_collection(
    name="my_documents",
    embedding_function=openai_ef
)

# Add documents
collection.add(
    documents=["Python is a programming language", "Dogs are mammals", "Paris is in France"],
    ids=["doc1", "doc2", "doc3"],
    metadatas=[{"source": "wiki"}, {"source": "wiki"}, {"source": "wiki"}]
)

# Query
results = collection.query(
    query_texts=["What programming languages exist?"],
    n_results=2
)

print(results["documents"])    # most similar docs
print(results["distances"])    # similarity scores
print(results["metadatas"])    # metadata for each result
```

### Pinecone (Best for production at scale)
- Fully managed cloud service
- Handles billions of vectors, auto-scaling

```bash
pip install pinecone
```

```python
from pinecone import Pinecone, ServerlessSpec

pc = Pinecone(api_key="your-pinecone-key")

# Create index
pc.create_index(
    name="my-index",
    dimension=1536,  # must match your embedding model's dimensions
    metric="cosine",
    spec=ServerlessSpec(cloud="aws", region="us-east-1")
)

index = pc.Index("my-index")

# Upsert vectors
vectors = [
    {"id": "doc1", "values": [0.1, 0.2, ...], "metadata": {"text": "...", "source": "wiki"}},
]
index.upsert(vectors=vectors)

# Query
results = index.query(
    vector=[0.1, 0.2, ...],  # embed your query first
    top_k=5,
    include_metadata=True
)
```

### Weaviate
- Open source, can self-host or use cloud
- Built-in hybrid search (vector + keyword)
- GraphQL API

### pgvector (Best if you already use PostgreSQL)
- PostgreSQL extension — store vectors in your existing database
- No new infrastructure needed

```sql
-- Install extension
CREATE EXTENSION vector;

-- Create table with vector column
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(1536),
    metadata JSONB
);

-- Create index for fast search
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops);

-- Search
SELECT content, 1 - (embedding <=> '[0.1,0.2,...]'::vector) AS similarity
FROM documents
ORDER BY embedding <=> '[0.1,0.2,...]'::vector
LIMIT 5;
```

```bash
pip install psycopg2-binary pgvector
```

---

## Choosing a Vector Database

| | Chroma | Pinecone | pgvector | Weaviate |
|-|--------|----------|----------|---------|
| Setup | Zero | Cloud signup | PostgreSQL | Docker/Cloud |
| Scale | Small/Medium | Enterprise | Medium | Large |
| Cost | Free | Paid | Free | Free/Paid |
| Hosting | Local/Cloud | Cloud only | Self/Cloud | Self/Cloud |
| Best for | Dev/prototypes | Production | Existing PG users | Hybrid search |

---

## Key Concepts

| Term | Meaning |
|------|---------|
| Index / Collection | Container for vectors |
| Dimension | Number of values in each vector (must match embedding model) |
| Upsert | Insert or update a vector |
| Top-K | Return K most similar results |
| Metadata filtering | Filter results by tags/attributes before similarity search |
| HNSW | Approximate nearest neighbor algorithm (fast but approximate) |
| IVFFlat | Another ANN algorithm used by pgvector |
