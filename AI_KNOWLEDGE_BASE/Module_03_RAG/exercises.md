# Module 03 — Exercises

## Exercise 1: Embed and Visualize
Take 20 short sentences (mix of topics: food, sports, tech, animals). Embed them all, then use PCA to reduce to 2D and plot them. Sentences about similar topics should cluster together.

```python
from sklearn.decomposition import PCA
import matplotlib.pyplot as plt
from openai import OpenAI

sentences = [
    "Python is a programming language",
    "JavaScript runs in browsers",
    "Soccer is played with a round ball",
    "Basketball requires a hoop",
    "Pizza is made with dough and tomato sauce",
    # ... add 15 more covering ~4 topics
]

def embed_all(texts: list) -> list:
    # Embed all texts in one API call
    pass

def visualize_clusters(sentences, embeddings):
    # PCA to 2D, then scatter plot
    # Label each point with the first 3 words of the sentence
    pass
```

**Goal**: See with your own eyes that embeddings capture meaning.

---

## Exercise 2: Chunking Comparison
Take a long article (find one on Wikipedia, copy the text). Apply 3 different chunking strategies:
1. Fixed size (200 chars, 50 overlap)
2. By paragraph
3. Recursive character splitter (LangChain)

For each strategy, print:
- Number of chunks
- Average chunk size
- Min/max chunk size
- First 3 chunks (preview)

```python
text = open("article.txt").read()

strategies = {
    "fixed": lambda t: chunk_fixed(t, 200, 50),
    "paragraph": lambda t: chunk_by_paragraph(t),
    "recursive": lambda t: recursive_split(t, 200, 50)
}

for name, fn in strategies.items():
    chunks = fn(text)
    # print stats
```

**Goal**: Understand the practical differences between chunking approaches.

---

## Exercise 3: Semantic Search from Scratch
Without using any vector database, implement semantic search over 50 Wikipedia article summaries using only NumPy and the embeddings API.

```python
import numpy as np

# 1. Fetch 50 article summaries (use wikipedia-api or just hardcode them)
# 2. Embed all of them
# 3. Implement cosine similarity search
# 4. Given a query, return top 5 most relevant

def cosine_similarity(a, b): ...
def search(query: str, corpus: list, embeddings: list, top_k=5): ...

# Test queries:
queries = [
    "history of space exploration",
    "how does the human heart work",
    "famous painters in Europe",
]
```

---

## Exercise 4: Retrieval Quality Tester
Build an evaluation harness for your retrieval:
- Index 10 documents
- Create 5 question-answer pairs where you know which document has the answer
- Measure: for each question, was the correct document in the top-3 retrieved?
- Report: recall@3 score

```python
test_cases = [
    {
        "question": "What is the refund policy?",
        "expected_source": "refund_policy.txt"
    },
    # ...
]

def evaluate_retrieval(test_cases: list) -> float:
    hits = 0
    for case in test_cases:
        results = retrieve(case["question"], top_k=3)
        sources = [r["source"] for r in results]
        if case["expected_source"] in sources:
            hits += 1
    return hits / len(test_cases)  # recall@3

score = evaluate_retrieval(test_cases)
print(f"Recall@3: {score:.0%}")
```

---

## Exercise 5: Hybrid Search
Implement a hybrid search that combines:
- Vector similarity (semantic)
- BM25 keyword search

Use Reciprocal Rank Fusion (RRF) to combine the scores.

```python
from rank_bm25 import BM25Okapi  # pip install rank-bm25

def hybrid_search(query: str, documents: list, top_k=5) -> list:
    # 1. Semantic search → ranked list
    # 2. BM25 search → ranked list
    # 3. RRF fusion:
    #    rrf_score = sum(1 / (rank + 60) for each list)
    # 4. Return top_k by RRF score
    pass
```
