# Module 06 — Exercises

## Exercise 1: Write an Eval Suite
Create a test suite for a customer support chatbot. Write 10 test cases covering:
- Correct answers to product questions
- Refusing off-topic questions
- Handling rude input politely
- Not making up prices or policies

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class TestCase:
    name: str
    input: str
    must_contain: list[str] = None        # at least one must appear
    must_not_contain: list[str] = None    # none of these should appear
    llm_judge_criteria: str = None        # description for LLM judge

test_suite = [
    TestCase(
        name="basic_product_question",
        input="What are your business hours?",
        must_contain=["9", "5", "Monday"],  # should mention hours
        must_not_contain=["I don't know", "sorry, I can't"]
    ),
    TestCase(
        name="off_topic_refusal",
        input="What's the best pizza recipe?",
        llm_judge_criteria="Response should politely redirect to business topics, not answer the question"
    ),
    # ... 8 more
]

def run_eval(test_suite: list[TestCase]) -> dict:
    ...
```

---

## Exercise 2: LLM-as-Judge Scorer
Build a function that takes any question + answer pair and scores it on 4 dimensions (1-5 each):
- Accuracy
- Clarity
- Completeness
- Conciseness

Then run it on 5 answers generated at different temperatures (0, 0.3, 0.7, 1.0, 1.5) and see which temperature gives the best average score.

```python
def score_response(question: str, answer: str) -> dict:
    # Returns {"accuracy": 4, "clarity": 5, "completeness": 3, "conciseness": 4, "average": 4.0}
    ...

# Run experiment
question = "Explain how a neural network learns"
temperatures = [0, 0.3, 0.7, 1.0, 1.5]
results = {}
for temp in temperatures:
    answer = generate(question, temperature=temp)
    results[temp] = score_response(question, answer)

# Print comparison table
```

---

## Exercise 3: Add Logging to Module 05 Project
Take the web app from Module 05 and add structured logging:
- Log every request: user message (truncated to 100 chars), session ID, timestamp
- Log every LLM call: model, tokens in/out, latency, cost
- Log every retrieval: query, number of chunks, top similarity score
- Store logs in SQLite
- Add a `/stats` endpoint that returns: total calls, total tokens, avg latency, total cost

```python
# Create a Logger class that wraps your existing calls
class AILogger:
    def __init__(self, db_path: str = "logs.db"):
        ...

    def log_request(self, session_id, user_message): ...
    def log_llm_call(self, model, input_tokens, output_tokens, latency_ms): ...
    def log_retrieval(self, query, num_chunks, top_score): ...
    def get_stats(self) -> dict: ...
```

---

## Exercise 4: Rate Limiter + Abuse Detection
Implement a production-grade rate limiter that:
- Allows 10 requests per minute per user
- Detects "abuse" patterns: same message sent 3+ times, messages >2000 chars, >50 messages in 10 minutes
- Logs abuse events separately
- Returns informative error messages: "Slow down! You've hit the rate limit. Try again in 45 seconds."

```python
import time
from collections import defaultdict, deque

class RateLimiter:
    def __init__(self):
        self.user_windows = defaultdict(deque)     # sliding window
        self.user_messages = defaultdict(list)     # for pattern detection
        self.abuse_log = []

    def check(self, user_id: str, message: str) -> tuple[bool, str]:
        # Returns (allowed, reason_if_blocked)
        ...

    def _detect_abuse(self, user_id: str, message: str) -> Optional[str]:
        # Returns abuse reason or None
        ...
```

---

## Exercise 5: A/B Test Two Prompts
Build a simple A/B testing framework that:
- Routes 50% of traffic to prompt A, 50% to prompt B
- Records which prompt was used + the LLM judge score for each response
- After 20 samples, reports a winner with statistical reasoning

```python
import random

PROMPT_A = "You are a helpful assistant. Be concise."
PROMPT_B = "You are an expert assistant. Think carefully before responding. Be thorough."

class ABTest:
    def __init__(self):
        self.results = {"A": [], "B": []}

    def get_response(self, question: str) -> tuple[str, str]:
        # Returns (response, variant)
        variant = random.choice(["A", "B"])
        system = PROMPT_A if variant == "A" else PROMPT_B
        response = call_llm(question, system=system)
        return response, variant

    def record_score(self, variant: str, score: float):
        self.results[variant].append(score)

    def report(self):
        for variant, scores in self.results.items():
            avg = sum(scores) / len(scores) if scores else 0
            print(f"Variant {variant}: {avg:.2f} avg score ({len(scores)} samples)")
```
