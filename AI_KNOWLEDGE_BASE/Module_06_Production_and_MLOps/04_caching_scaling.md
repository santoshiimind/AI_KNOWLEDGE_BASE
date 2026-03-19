# Caching, Latency & Scaling

## Sources of Latency in AI Apps

```
User Request
    → Network to API server      (10-50ms)
    → API queueing               (0-500ms)
    → Token processing (TTFT)    (200-2000ms)  ← First token latency
    → Token generation           (varies with output length)
    → Response back to user      (10-50ms)
```

**TTFT** (Time To First Token) is the most important UX metric.

---

## Caching Strategies

### 1. Semantic Cache (Cache Similar Questions)

Cache responses for semantically similar queries:

```python
import numpy as np
from openai import OpenAI

openai_client = OpenAI()

class SemanticCache:
    def __init__(self, similarity_threshold: float = 0.95):
        self.threshold = similarity_threshold
        self.cache = []  # list of (embedding, question, answer)

    def _embed(self, text: str) -> list[float]:
        response = openai_client.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        return response.data[0].embedding

    def _similarity(self, a, b) -> float:
        a, b = np.array(a), np.array(b)
        return float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))

    def get(self, question: str) -> str | None:
        if not self.cache:
            return None
        query_emb = self._embed(question)
        for emb, _, answer in self.cache:
            if self._similarity(query_emb, emb) >= self.threshold:
                return answer
        return None

    def set(self, question: str, answer: str):
        emb = self._embed(question)
        self.cache.append((emb, question, answer))

cache = SemanticCache(similarity_threshold=0.93)

def cached_llm_call(question: str) -> str:
    # Check cache first
    cached = cache.get(question)
    if cached:
        print("[CACHE HIT]")
        return cached

    # Call the model
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        messages=[{"role": "user", "content": question}]
    )
    answer = response.content[0].text

    # Cache for next time
    cache.set(question, answer)
    return answer
```

### 2. Exact Cache with Redis

For high-traffic apps, use Redis as a distributed cache:

```python
import redis
import hashlib
import json
import anthropic

r = redis.Redis(host="localhost", port=6379, decode_responses=True)
client = anthropic.Anthropic()

def cache_key(messages: list, system: str) -> str:
    content = json.dumps({"messages": messages, "system": system}, sort_keys=True)
    return f"llm:{hashlib.sha256(content.encode()).hexdigest()}"

def cached_call(messages: list, system: str = "", ttl_seconds: int = 3600) -> str:
    key = cache_key(messages, system)

    # Check cache
    cached = r.get(key)
    if cached:
        return json.loads(cached)

    # Call model
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=system,
        messages=messages
    )
    answer = response.content[0].text

    # Store with TTL
    r.setex(key, ttl_seconds, json.dumps(answer))
    return answer
```

### 3. Anthropic Prompt Caching (Native)

Cache large system prompts to save 90% on input tokens:

```python
import anthropic

client = anthropic.Anthropic()

LARGE_SYSTEM_CONTEXT = "..." * 1000  # 10K+ character knowledge base

def call_with_prompt_cache(question: str) -> str:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=[
            {
                "type": "text",
                "text": LARGE_SYSTEM_CONTEXT,
                "cache_control": {"type": "ephemeral"}  # cache this block
            }
        ],
        messages=[{"role": "user", "content": question}]
    )

    # Check cache stats
    print(f"Cache created: {response.usage.cache_creation_input_tokens}")
    print(f"Cache read: {response.usage.cache_read_input_tokens}")

    return response.content[0].text
```

---

## Reducing Latency

### Parallel Processing

```python
import asyncio
import anthropic

async_client = anthropic.AsyncAnthropic()

async def parallel_tasks(tasks: list[str]) -> list[str]:
    """Run multiple LLM tasks in parallel."""
    async def run_task(task):
        response = await async_client.messages.create(
            model="claude-haiku-4-5-20251001",  # fast model
            max_tokens=512,
            messages=[{"role": "user", "content": task}]
        )
        return response.content[0].text

    results = await asyncio.gather(*[run_task(t) for t in tasks])
    return list(results)

# Run 5 tasks concurrently instead of sequentially
tasks = ["Summarize topic A", "Summarize topic B", "Summarize topic C"]
results = asyncio.run(parallel_tasks(tasks))
```

### Streaming for Perceived Speed

```python
# Without streaming: user waits 3s for full response
# With streaming: user sees first word in 0.3s

with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": prompt}]
) as stream:
    for text in stream.text_stream:
        yield text  # send each token immediately
```

### Choose Faster Models for Latency-Critical Paths

```python
# Routing strategy: use fast model for simple tasks, slow model for complex
def smart_route(question: str) -> str:
    is_simple = len(question) < 100 and "?" in question

    model = "claude-haiku-4-5-20251001" if is_simple else "claude-sonnet-4-6"
    # Haiku: ~0.5s TTFT | Sonnet: ~1-2s TTFT

    response = client.messages.create(
        model=model,
        max_tokens=512,
        messages=[{"role": "user", "content": question}]
    )
    return response.content[0].text
```

---

## Scaling Patterns

### Queue-Based Processing (Celery)

```python
from celery import Celery

app = Celery("ai_tasks", broker="redis://localhost/0", backend="redis://localhost/0")

@app.task(bind=True, max_retries=3)
def process_document(self, doc_id: str, content: str):
    try:
        result = analyze_document(content)
        save_result(doc_id, result)
        return result
    except Exception as exc:
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)

# Submit job and return immediately
task = process_document.delay(doc_id, content)
# Poll for result later
result = task.get(timeout=60)
```

### Horizontal Scaling

```
Load Balancer
     ├── App Instance 1  (FastAPI + LLM client)
     ├── App Instance 2
     └── App Instance 3
              ↓
         Redis Cache
              ↓
         LLM API (rate limited)
```

---

## Performance Budget

| Target | Acceptable | Needs Work |
|--------|-----------|------------|
| TTFT | < 500ms | > 2s |
| Total response | < 3s for short | > 10s |
| Cache hit rate | > 20% | < 5% |
| Error rate | < 0.5% | > 2% |
| Cost per query | Set per use case | Track daily |
