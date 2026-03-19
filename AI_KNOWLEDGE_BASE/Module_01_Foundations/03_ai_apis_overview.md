# AI APIs Overview

## Major API Providers

### Anthropic — Claude
- **Models**: Claude 3.5 Haiku, Claude 3.5 Sonnet, Claude Opus 4
- **Strengths**: Long context (200K), strong reasoning, safety, coding, following complex instructions
- **SDK**: `pip install anthropic`
- **Docs**: https://docs.anthropic.com

### OpenAI — GPT
- **Models**: GPT-4o, GPT-4o-mini, o1, o3
- **Strengths**: Widest ecosystem, most integrations, strong multimodal
- **SDK**: `pip install openai`
- **Docs**: https://platform.openai.com/docs

### Google — Gemini
- **Models**: Gemini 1.5 Pro, Gemini 1.5 Flash, Gemini 2.0
- **Strengths**: 1M+ context window, multimodal (video, audio), Google ecosystem
- **SDK**: `pip install google-generativeai`
- **Docs**: https://ai.google.dev

### Meta — Llama (Open Source)
- **Models**: Llama 3.1, Llama 3.2
- **Strengths**: Free, self-hostable, customizable
- **Run locally**: Ollama, vLLM, LM Studio

### Mistral AI
- **Models**: Mistral Large, Mistral 7B
- **Strengths**: Efficient, open weights, European data residency
- **SDK**: `pip install mistralai`

---

## API Concepts Common Across Providers

### Messages Format
Most modern LLM APIs use a **messages array** format:

```python
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is the capital of France?"},
    {"role": "assistant", "content": "The capital of France is Paris."},
    {"role": "user", "content": "What is its population?"}
]
```

### Key Parameters

| Parameter | Description | Typical Range |
|-----------|-------------|---------------|
| `model` | Which model to use | varies by provider |
| `max_tokens` | Max tokens in response | 1 – 128000 |
| `temperature` | Randomness (0 = deterministic) | 0.0 – 2.0 |
| `top_p` | Nucleus sampling | 0.0 – 1.0 |
| `stream` | Stream response token by token | true/false |
| `stop` | Stop generation at this string | any string |

### Response Structure
```json
{
  "id": "msg_abc123",
  "model": "claude-sonnet-4-6",
  "usage": {
    "input_tokens": 25,
    "output_tokens": 142
  },
  "content": [
    {
      "type": "text",
      "text": "The capital of France is Paris..."
    }
  ]
}
```

---

## Quick Comparison

| Feature | Anthropic | OpenAI | Google |
|---------|-----------|--------|--------|
| Context Window | 200K | 128K | 1M+ |
| Function Calling | Yes | Yes | Yes |
| Vision | Yes | Yes | Yes |
| Streaming | Yes | Yes | Yes |
| Fine-tuning | Limited | Yes | Yes |
| Local/On-prem | No | No | Vertex AI |
| Open Source | No | No | No |

---

## Choosing the Right API

- **Building a new app?** → Start with Claude Sonnet or GPT-4o-mini (cost-effective, capable)
- **Need long documents?** → Claude (200K context) or Gemini (1M+ context)
- **Need multimodal (video)?** → Gemini
- **Need maximum ecosystem/integrations?** → OpenAI
- **Privacy/self-hosting?** → Llama + Ollama
- **Production at scale?** → Consider cost per token carefully

---

## Cost Awareness

Pricing is per **1M tokens** (as of early 2025, approximate):

| Model | Input | Output |
|-------|-------|--------|
| GPT-4o | $2.50 | $10.00 |
| GPT-4o-mini | $0.15 | $0.60 |
| Claude 3.5 Sonnet | $3.00 | $15.00 |
| Claude 3.5 Haiku | $0.80 | $4.00 |
| Gemini 1.5 Flash | $0.075 | $0.30 |

> Always check current pricing on provider docs. Prices drop frequently.

---

## Hello World — Anthropic Claude

```python
import anthropic

client = anthropic.Anthropic(api_key="your-api-key")

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Hello, Claude!"}
    ]
)

print(message.content[0].text)
```

## Hello World — OpenAI

```python
from openai import OpenAI

client = OpenAI(api_key="your-api-key")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "Hello, GPT!"}
    ]
)

print(response.choices[0].message.content)
```

---

## Resources

- [Anthropic API Docs](https://docs.anthropic.com)
- [OpenAI API Docs](https://platform.openai.com/docs)
- [Google AI Studio](https://aistudio.google.com)
- [Together AI](https://www.together.ai) — run open-source models via API
