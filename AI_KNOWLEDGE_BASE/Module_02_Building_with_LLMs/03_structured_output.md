# Structured Output / JSON Mode

## Why Structured Output?

When building applications, you need predictable, parseable responses — not free-form text.
Instead of: `"The user's name is John and he is 30 years old."`
You want: `{"name": "John", "age": 30}`

---

## Approach 1: Instruct in Prompt

The simplest approach — tell the model to return JSON.

```python
import anthropic
import json

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": """
        Extract the following information from this text and return as JSON only.
        No explanation, just the JSON object.

        Schema:
        {
            "name": string,
            "age": number,
            "email": string | null
        }

        Text: "Hi I'm Sarah, I'm 28 years old. You can reach me at sarah@email.com"
        """
    }]
)

data = json.loads(response.content[0].text)
print(data["name"])  # Sarah
```

---

## Approach 2: Prefill the Response (Anthropic)

Force the model to start its response with `{` to guarantee JSON output:

```python
import anthropic
import json

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": "Extract: name, sentiment (positive/negative/neutral), and key topics from: 'Claude is an amazing AI assistant! I love how it handles complex coding tasks.'"
        },
        {
            "role": "assistant",
            "content": "{"  # prefill forces JSON response
        }
    ]
)

# Prepend the "{" we used as prefill
raw = "{" + response.content[0].text
data = json.loads(raw)
print(data)
```

---

## Approach 3: Pydantic Validation

Use Pydantic to validate and parse the output:

```python
from pydantic import BaseModel, ValidationError
from typing import Optional, List
import anthropic
import json

client = anthropic.Anthropic()

class ProductReview(BaseModel):
    product_name: str
    rating: int  # 1-5
    sentiment: str  # positive, negative, neutral
    pros: List[str]
    cons: List[str]
    summary: str

def extract_review(review_text: str) -> ProductReview:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=f"""Extract review information and return as JSON matching this schema:
        {ProductReview.model_json_schema()}
        Return only valid JSON, no other text.""",
        messages=[{"role": "user", "content": review_text}]
    )

    try:
        data = json.loads(response.content[0].text)
        return ProductReview(**data)
    except (json.JSONDecodeError, ValidationError) as e:
        print(f"Parsing failed: {e}")
        raise

review = extract_review("""
    The AirPods Pro are fantastic! Great sound quality and the noise cancellation
    is top notch. Battery life is decent. However, they're expensive and the case
    scratches easily. Overall I'd give them 4 out of 5.
""")

print(review.product_name)  # AirPods Pro
print(review.rating)        # 4
print(review.pros)          # ['Great sound quality', 'Top notch noise cancellation', ...]
```

---

## Approach 4: OpenAI Structured Outputs (JSON Schema)

OpenAI has built-in strict JSON mode:

```python
from openai import OpenAI
from pydantic import BaseModel

client = OpenAI()

class CalendarEvent(BaseModel):
    name: str
    date: str
    participants: list[str]

completion = client.beta.chat.completions.parse(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "Extract the event information."},
        {"role": "user", "content": "Alice and Bob are meeting on March 20th to discuss the Q2 roadmap."}
    ],
    response_format=CalendarEvent,
)

event = completion.choices[0].message.parsed
print(event.name)          # Q2 roadmap discussion
print(event.date)          # March 20th
print(event.participants)  # ['Alice', 'Bob']
```

---

## Handling Failures

```python
import json
import re

def safe_parse_json(text: str) -> dict:
    # Try direct parse first
    try:
        return json.loads(text)
    except json.JSONDecodeError:
        pass

    # Try to extract JSON from markdown code blocks
    match = re.search(r'```(?:json)?\s*(\{.*?\})\s*```', text, re.DOTALL)
    if match:
        try:
            return json.loads(match.group(1))
        except json.JSONDecodeError:
            pass

    # Try to find any JSON object in the text
    match = re.search(r'\{.*\}', text, re.DOTALL)
    if match:
        try:
            return json.loads(match.group(0))
        except json.JSONDecodeError:
            pass

    raise ValueError(f"Could not parse JSON from: {text[:200]}")
```

---

## Best Practices

1. Provide the exact schema in the prompt (JSON Schema or Pydantic model)
2. Use prefilling (Anthropic) or JSON mode (OpenAI) for reliability
3. Always validate with Pydantic before using the data
4. Have a fallback/retry strategy for parse failures
5. Keep schemas flat when possible — deep nesting increases error rate
