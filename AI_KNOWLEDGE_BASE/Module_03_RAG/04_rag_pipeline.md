# Building an End-to-End RAG Pipeline

## Full Pipeline Overview

```
INDEXING (one-time):
Documents → Load → Chunk → Embed → Store in Vector DB

QUERYING (per request):
User Query → Embed Query → Retrieve Top-K Chunks → Build Prompt → LLM → Answer
```

---

## Complete RAG Implementation

```python
import os
import anthropic
from openai import OpenAI
import chromadb
from chromadb.utils import embedding_functions

# ---- SETUP ----
anthropic_client = anthropic.Anthropic()
openai_client = OpenAI()

chroma_client = chromadb.PersistentClient(path="./rag_db")

openai_ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key=os.environ["OPENAI_API_KEY"],
    model_name="text-embedding-3-small"
)

collection = chroma_client.get_or_create_collection(
    name="knowledge_base",
    embedding_function=openai_ef
)


# ---- INDEXING ----
def load_and_index(file_path: str):
    with open(file_path, "r", encoding="utf-8") as f:
        text = f.read()

    # Chunk the document
    chunks = chunk_text(text, chunk_size=500, overlap=50)

    # Store in vector DB (Chroma handles embedding automatically)
    collection.add(
        documents=chunks,
        ids=[f"{file_path}_{i}" for i in range(len(chunks))],
        metadatas=[{"source": file_path, "chunk": i} for i in range(len(chunks))]
    )
    print(f"Indexed {len(chunks)} chunks from {file_path}")


def chunk_text(text: str, chunk_size: int = 500, overlap: int = 50) -> list[str]:
    chunks = []
    start = 0
    while start < len(text):
        end = min(start + chunk_size, len(text))
        # Extend to nearest sentence end
        if end < len(text):
            period_pos = text.rfind('.', start, end)
            if period_pos > start:
                end = period_pos + 1
        chunks.append(text[start:end].strip())
        start += chunk_size - overlap
    return [c for c in chunks if c]


# ---- RETRIEVAL ----
def retrieve(query: str, top_k: int = 5) -> list[dict]:
    results = collection.query(
        query_texts=[query],
        n_results=top_k,
        include=["documents", "metadatas", "distances"]
    )

    retrieved = []
    for doc, meta, dist in zip(
        results["documents"][0],
        results["metadatas"][0],
        results["distances"][0]
    ):
        retrieved.append({
            "text": doc,
            "source": meta["source"],
            "similarity": 1 - dist  # convert distance to similarity
        })

    return retrieved


# ---- GENERATION ----
def answer_question(question: str) -> dict:
    # Retrieve relevant chunks
    chunks = retrieve(question, top_k=5)

    # Filter low-relevance chunks
    relevant_chunks = [c for c in chunks if c["similarity"] > 0.5]

    if not relevant_chunks:
        return {
            "answer": "I couldn't find relevant information to answer your question.",
            "sources": []
        }

    # Build context
    context = "\n\n---\n\n".join([
        f"Source: {c['source']}\n{c['text']}"
        for c in relevant_chunks
    ])

    # Generate answer
    response = anthropic_client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system="""You are a helpful assistant that answers questions based on provided context.

Rules:
- Only use information from the provided context
- If the context doesn't contain the answer, say so clearly
- Always cite your sources
- Be concise and accurate""",
        messages=[{
            "role": "user",
            "content": f"""Context:
{context}

Question: {question}

Answer the question based only on the context above."""
        }]
    )

    return {
        "answer": response.content[0].text,
        "sources": list(set(c["source"] for c in relevant_chunks)),
        "chunks_used": len(relevant_chunks)
    }


# ---- USAGE ----
# Index your documents
# load_and_index("company_handbook.txt")
# load_and_index("product_docs.txt")

# Query
result = answer_question("What is our vacation policy?")
print(result["answer"])
print(f"Sources: {result['sources']}")
```

---

## RAG with LangChain (Higher-Level)

```python
from langchain_community.document_loaders import DirectoryLoader, TextLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 1. Load documents
loader = DirectoryLoader("./docs", glob="**/*.txt", loader_cls=TextLoader)
documents = loader.load()

# 2. Split
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(documents)

# 3. Embed and store
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=OpenAIEmbeddings(model="text-embedding-3-small"),
    persist_directory="./chroma_db"
)

# 4. Create retriever
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# 5. Build RAG chain
template = """Answer the question based only on the following context:

{context}

Question: {question}
"""

prompt = ChatPromptTemplate.from_template(template)
llm = ChatOpenAI(model="gpt-4o")

rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# 6. Query
answer = rag_chain.invoke("What is our refund policy?")
print(answer)
```

---

## Common RAG Problems & Fixes

| Problem | Symptoms | Fix |
|---------|----------|-----|
| Poor retrieval | Wrong chunks returned | Better chunking, re-rank results |
| Hallucination | LLM ignores context | Stronger system prompt, reduce temperature |
| Missing context | Answer is incomplete | Increase top_k, fix chunk size |
| Too much noise | Irrelevant info in context | Filter by similarity score, use metadata filters |
| Slow queries | High latency | Cache embeddings, use faster models |
| Stale data | Outdated answers | Re-index on document update |

---

## Advanced RAG Techniques

### 1. Re-ranking
After retrieval, use a cross-encoder to re-rank results by relevance:

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

def rerank(query: str, chunks: list[str], top_k: int = 3) -> list[str]:
    pairs = [(query, chunk) for chunk in chunks]
    scores = reranker.predict(pairs)
    ranked = sorted(zip(scores, chunks), reverse=True)
    return [chunk for _, chunk in ranked[:top_k]]
```

### 2. HyDE (Hypothetical Document Embeddings)
Generate a hypothetical answer to improve retrieval:

```python
def hyde_retrieve(question: str) -> list[dict]:
    # Generate a hypothetical answer
    hypothesis = anthropic_client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=200,
        messages=[{"role": "user", "content": f"Write a short answer to: {question}"}]
    ).content[0].text

    # Use the hypothesis to search instead of the raw question
    return retrieve(hypothesis, top_k=5)
```

### 3. Query Expansion
Generate multiple versions of the query:

```python
def expand_query(question: str) -> list[str]:
    response = anthropic_client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=200,
        messages=[{
            "role": "user",
            "content": f"Generate 3 different ways to ask this question. Return as JSON array.\nQuestion: {question}"
        }]
    )
    import json
    queries = json.loads(response.content[0].text)
    return [question] + queries  # original + variations
```
