# Token Management & Cost Optimization

## Understanding Token Costs

Every API call costs money based on tokens:
- **Input tokens**: your prompt + conversation history + system prompt
- **Output tokens**: the model's response (usually 3-5x more expensive)

```
Cost = (input_tokens × input_price) + (output_tokens × output_price)
```

---

## Counting Tokens Before Sending

```python
import anthropic

client = anthropic.Anthropic()

# Count tokens without making a full request
response = client.messages.count_tokens(
    model="claude-sonnet-4-6",
    system="You are a helpful assistant.",
    messages=[{"role": "user", "content": "Explain quantum computing in detail."}]
)

print(f"This request will use ~{response.input_tokens} input tokens")
```

For OpenAI, use the `tiktoken` library:
```python
import tiktoken

enc = tiktoken.encoding_for_model("gpt-4o")
tokens = enc.encode("Hello, how are you?")
print(f"Token count: {len(tokens)}")
```

---

## Strategies to Reduce Costs

### 1. Choose the Right Model
Don't use expensive models for simple tasks:

| Task | Recommended Model |
|------|------------------|
| Simple Q&A, classification | Claude Haiku, GPT-4o-mini |
| Complex reasoning, coding | Claude Sonnet, GPT-4o |
| Most demanding tasks | Claude Opus |

```python
def get_model_for_task(task_type: str) -> str:
    if task_type in ["classification", "extraction", "simple_qa"]:
        return "claude-haiku-4-5-20251001"  # ~10x cheaper
    elif task_type in ["coding", "analysis", "writing"]:
        return "claude-sonnet-4-6"
    else:
        return "claude-opus-4-6"
```

### 2. Trim Conversation History

Long conversation histories consume input tokens. Trim them:

```python
def trim_history(messages: list, max_tokens: int = 4000) -> list:
    """Keep only the most recent messages within token budget."""
    # Always keep system + first user message for context
    trimmed = []
    token_count = 0

    for message in reversed(messages):
        # Rough estimate: 1 token ≈ 4 chars
        msg_tokens = len(str(message["content"])) // 4
        if token_count + msg_tokens > max_tokens:
            break
        trimmed.insert(0, message)
        token_count += msg_tokens

    return trimmed
```

### 3. Summarize Long Conversations

```python
def summarize_old_messages(messages: list, keep_recent: int = 4) -> list:
    if len(messages) <= keep_recent:
        return messages

    old_messages = messages[:-keep_recent]
    recent_messages = messages[-keep_recent:]

    # Summarize old messages
    summary_response = client.messages.create(
        model="claude-haiku-4-5-20251001",  # use cheap model for summarization
        max_tokens=300,
        messages=[{
            "role": "user",
            "content": f"Summarize this conversation in 3-4 sentences:\n{old_messages}"
        }]
    )

    summary = summary_response.content[0].text
    summary_message = {
        "role": "user",
        "content": f"[Previous conversation summary: {summary}]"
    }

    return [summary_message] + recent_messages
```

### 4. Cache System Prompts (Anthropic Prompt Caching)

If you have a large system prompt used in many requests, enable caching:

```python
import anthropic

client = anthropic.Anthropic()

# Large system prompt (>1024 tokens) — mark for caching
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "You are an expert assistant with the following knowledge base:\n" + large_knowledge_text,
            "cache_control": {"type": "ephemeral"}  # cache this block
        }
    ],
    messages=[{"role": "user", "content": "What does the document say about X?"}]
)

# Cached tokens cost ~90% less on repeated calls
print(response.usage.cache_read_input_tokens)   # tokens read from cache
print(response.usage.cache_creation_input_tokens)  # tokens written to cache
```

### 5. Compress Your Prompts

- Remove unnecessary whitespace and filler words
- Use abbreviations in few-shot examples
- Use structured formats (JSON/YAML) instead of verbose prose

```python
# Verbose (wasteful):
"""
Please carefully analyze the following piece of text and determine what
the overall sentiment is. The sentiment should be classified as either
positive, negative, or neutral based on the emotional tone of the text.
"""

# Compressed (same result, fewer tokens):
"""Classify sentiment as positive/negative/neutral:"""
```

### 6. Set Appropriate max_tokens

Only request as many output tokens as you need:

```python
# For classification (one word answer)
response = client.messages.create(max_tokens=10, ...)

# For a short summary
response = client.messages.create(max_tokens=200, ...)

# For long-form content
response = client.messages.create(max_tokens=2000, ...)
```

---

## Cost Tracking

```python
import anthropic
from dataclasses import dataclass, field

@dataclass
class CostTracker:
    total_input_tokens: int = 0
    total_output_tokens: int = 0
    total_cost_usd: float = 0.0

    # Claude Sonnet pricing (per 1M tokens)
    input_price_per_m: float = 3.00
    output_price_per_m: float = 15.00

    def add_usage(self, usage):
        self.total_input_tokens += usage.input_tokens
        self.total_output_tokens += usage.output_tokens
        cost = (
            (usage.input_tokens / 1_000_000) * self.input_price_per_m +
            (usage.output_tokens / 1_000_000) * self.output_price_per_m
        )
        self.total_cost_usd += cost

    def report(self):
        print(f"Total tokens: {self.total_input_tokens:,} in / {self.total_output_tokens:,} out")
        print(f"Estimated cost: ${self.total_cost_usd:.4f}")

tracker = CostTracker()

response = client.messages.create(...)
tracker.add_usage(response.usage)
tracker.report()
```

---

## Key Rules of Thumb

- Output tokens cost 3-5x more than input tokens — keep responses focused
- A 200K context window ≠ you should always use it — long contexts are expensive
- Use cheap models (Haiku, GPT-4o-mini) for: classification, routing, summarization
- Use expensive models only where quality matters
- Prompt caching gives ~90% discount on repeated static content (Anthropic)
