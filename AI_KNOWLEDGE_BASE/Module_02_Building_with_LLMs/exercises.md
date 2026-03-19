# Module 02 — Exercises

## Exercise 1: Retry with Exponential Backoff
Write a `robust_call()` function that:
- Retries on rate limit errors (up to 5 times)
- Uses exponential backoff (1s, 2s, 4s, 8s, 16s)
- Raises immediately on non-retryable errors (4xx except 429)
- Logs each retry attempt with the wait time

```python
import anthropic
import time

def robust_call(messages: list, **kwargs) -> str:
    # Your code here
    pass

# Test by setting an artificially low rate limit or mocking the error
```

---

## Exercise 2: Streaming Progress Bar
Build a streaming response that shows:
- Tokens appearing word by word in the terminal
- A live token counter in the corner: `[47 tokens...]`
- Final summary: total tokens, total cost, time taken

```python
import anthropic
import time

def stream_with_stats(prompt: str):
    # Stream the response
    # Track: start_time, token_count
    # Print each token as it arrives
    # On completion, print a summary line
    pass
```

---

## Exercise 3: JSON Extractor
Given a messy paragraph of text, extract structured data into a Pydantic model. Handle the case where the model doesn't return valid JSON (retry once with a cleaner prompt).

```python
from pydantic import BaseModel
from typing import Optional, List

class JobPosting(BaseModel):
    company: str
    role: str
    location: str
    salary_min: Optional[int]
    salary_max: Optional[int]
    required_skills: List[str]
    remote: bool

raw_text = """
We're hiring at TechCorp! Looking for a Senior Python Developer to join our
London office (hybrid ok). Pay is £80k-£100k. Must know Python, FastAPI,
PostgreSQL, and Docker. React experience is a plus. Apply now!
"""

def extract_job_posting(text: str) -> JobPosting:
    # Your code here
    pass
```

---

## Exercise 4: Async Batch Processor
Process a list of 10 texts concurrently (classify each as positive/negative/neutral sentiment) and compare:
- Sequential time (one by one)
- Concurrent time (all at once with `asyncio.gather`)

```python
import asyncio
import time

texts = [
    "This product is amazing!",
    "Terrible experience, never buying again.",
    # ... 8 more
]

async def classify_sentiment(text: str) -> str:
    # Your code here
    pass

async def batch_classify(texts: list) -> list:
    # Use asyncio.gather
    pass

# Measure and compare times
```

---

## Exercise 5: Token Budget Manager
Write a `ConversationManager` class that:
- Keeps conversation history
- Before each request, checks if history exceeds 3000 tokens
- If over budget: summarizes oldest 50% of messages using a cheap model, replaces them with the summary
- Reports current token usage at each turn

```python
class ConversationManager:
    def __init__(self, token_budget: int = 3000):
        self.history = []
        self.token_budget = token_budget

    def add_and_respond(self, user_message: str) -> str:
        # Check budget, summarize if needed, then call model
        pass

    def estimate_tokens(self) -> int:
        pass
```
