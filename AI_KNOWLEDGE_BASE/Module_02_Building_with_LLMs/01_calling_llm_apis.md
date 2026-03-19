# Calling LLM APIs

## Setup

```bash
pip install anthropic openai python-dotenv
```

Store your keys safely:
```bash
# .env file (never commit this)
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
```

```python
from dotenv import load_dotenv
import os
load_dotenv()
```

---

## Basic Request — Anthropic

```python
import anthropic

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="You are a helpful coding assistant.",
    messages=[
        {"role": "user", "content": "Explain what a decorator is in Python."}
    ]
)

print(response.content[0].text)
print(f"Tokens used: {response.usage.input_tokens} in, {response.usage.output_tokens} out")
```

## Basic Request — OpenAI

```python
from openai import OpenAI

client = OpenAI()  # reads OPENAI_API_KEY from env

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful coding assistant."},
        {"role": "user", "content": "Explain what a decorator is in Python."}
    ]
)

print(response.choices[0].message.content)
```

---

## Multi-Turn Conversations

Maintain conversation history by appending messages:

```python
import anthropic

client = anthropic.Anthropic()
conversation_history = []

def chat(user_message):
    conversation_history.append({
        "role": "user",
        "content": user_message
    })

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system="You are a helpful assistant.",
        messages=conversation_history
    )

    assistant_message = response.content[0].text
    conversation_history.append({
        "role": "assistant",
        "content": assistant_message
    })

    return assistant_message

# Usage
print(chat("What is Python?"))
print(chat("What are its main use cases?"))  # model remembers context
```

---

## Error Handling & Retries

```python
import anthropic
import time
from anthropic import APIError, RateLimitError, APIConnectionError

client = anthropic.Anthropic()

def call_with_retry(messages, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = client.messages.create(
                model="claude-sonnet-4-6",
                max_tokens=1024,
                messages=messages
            )
            return response
        except RateLimitError:
            wait = 2 ** attempt  # exponential backoff: 1s, 2s, 4s
            print(f"Rate limited. Waiting {wait}s...")
            time.sleep(wait)
        except APIConnectionError as e:
            print(f"Connection error: {e}")
            time.sleep(1)
        except APIError as e:
            print(f"API error {e.status_code}: {e.message}")
            raise  # don't retry on other API errors

    raise Exception("Max retries exceeded")
```

---

## Async API Calls

For high-throughput apps, use async:

```python
import asyncio
import anthropic

client = anthropic.AsyncAnthropic()

async def call_llm(prompt):
    response = await client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.content[0].text

async def process_batch(prompts):
    tasks = [call_llm(p) for p in prompts]
    results = await asyncio.gather(*tasks)
    return results

# Run multiple prompts concurrently
prompts = ["Summarize AI", "Explain RAG", "What is fine-tuning?"]
results = asyncio.run(process_batch(prompts))
```

---

## Key Points

- Always load API keys from environment, never hardcode
- Track token usage in every response for cost monitoring
- Use exponential backoff for rate limit errors
- Use async clients for batch processing
- Keep conversation history on the client side — the API is stateless
