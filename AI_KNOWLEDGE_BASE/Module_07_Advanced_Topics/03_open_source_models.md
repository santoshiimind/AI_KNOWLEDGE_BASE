# Open-Source Models

## Why Open Source?

- **Free weights**: download and run anywhere
- **Privacy**: full data control
- **Customization**: fine-tune for your use case
- **No vendor lock-in**: switch models freely
- **Community**: vast ecosystem of fine-tunes and tools

---

## Major Model Families (2025)

### Meta Llama 3.x
The gold standard of open-source models.

| Model | Params | Context | Strengths |
|-------|--------|---------|-----------|
| Llama 3.2 3B | 3B | 128K | Edge devices, fast inference |
| Llama 3.2 11B | 11B | 128K | Multimodal (vision) |
| Llama 3.1 8B | 8B | 128K | General purpose |
| Llama 3.1 70B | 70B | 128K | Near GPT-4 quality |
| Llama 3.1 405B | 405B | 128K | Top open-source quality |

### Mistral AI
European alternative, known for efficiency.

| Model | Params | Strengths |
|-------|--------|-----------|
| Mistral 7B | 7B | Fast, efficient |
| Mixtral 8x7B | 47B (MoE) | High quality, efficient inference |
| Mistral Large | - | Enterprise quality |

### Microsoft Phi
Tiny but powerful "small language models" (SLMs).

| Model | Params | Strengths |
|-------|--------|-----------|
| Phi-3.5-mini | 3.8B | Excellent for size, coding |
| Phi-4 | 14B | State-of-the-art for size |

### Google Gemma
Open weights from Google.

| Model | Params | Strengths |
|-------|--------|-----------|
| Gemma 2 2B | 2B | Ultra lightweight |
| Gemma 2 9B | 9B | Balanced |
| Gemma 2 27B | 27B | High quality |

### Qwen (Alibaba)
Strong on code and multilingual tasks.

| Model | Strengths |
|-------|-----------|
| Qwen2.5 7B | Code, math |
| Qwen2.5-Coder 7B | Specialized for code |
| Qwen2.5 72B | Highest quality |

---

## Running Models via API (No Local Hardware)

Use cloud providers to access open-source models via API:

### Together AI

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.together.xyz/v1",
    api_key="your-together-api-key"
)

response = client.chat.completions.create(
    model="meta-llama/Meta-Llama-3.1-8B-Instruct-Turbo",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### Groq (Ultra-Fast Inference)

```python
from groq import Groq

client = Groq(api_key="your-groq-key")

response = client.chat.completions.create(
    model="llama-3.1-8b-instant",  # extremely fast
    messages=[{"role": "user", "content": "Hello!"}]
)
# Groq uses custom hardware (LPUs) for ~10x faster inference
```

### Hugging Face Inference API

```python
import requests

API_URL = "https://api-inference.huggingface.co/models/meta-llama/Llama-3.1-8B-Instruct"
headers = {"Authorization": f"Bearer {HF_TOKEN}"}

response = requests.post(API_URL, headers=headers, json={
    "inputs": "What is machine learning?",
    "parameters": {"max_new_tokens": 200}
})
```

---

## Running Locally with Hugging Face Transformers

```bash
pip install transformers accelerate torch
```

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

model_id = "microsoft/Phi-3.5-mini-instruct"  # small model good for testing

tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    torch_dtype=torch.float16,
    device_map="auto"  # auto-detect GPU/CPU
)

# Chat
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Explain how a transformer works."}
]

# Apply chat template
text = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs = tokenizer([text], return_tensors="pt").to(model.device)

with torch.no_grad():
    outputs = model.generate(**inputs, max_new_tokens=512, temperature=0.7)

response = tokenizer.decode(outputs[0][inputs.input_ids.shape[-1]:], skip_special_tokens=True)
print(response)
```

---

## Model Quantization

Reduce model size to run on consumer hardware:

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
import torch

# 4-bit quantization (4x smaller, ~10% quality loss)
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_use_double_quant=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3.1-8B",
    quantization_config=bnb_config,
    device_map="auto"
)
# 8B model: normally ~16GB → with 4-bit → ~5GB
```

---

## Choosing the Right Open-Source Model

```
Need fast/cheap → phi3.5, llama3.2:3b, mistral:7b
Need quality    → llama3.1:70b, Qwen2.5:72b
Need coding     → Qwen2.5-Coder, CodeLlama
Need multimodal → Llama 3.2 11B Vision
Need multilingual → Qwen2.5, Gemma 2
Need privacy    → Run locally with Ollama
Need scale      → Together AI, Groq, Fireworks
```

---

## Finding Models

- **Hugging Face Hub**: huggingface.co/models — largest catalog
- **Open LLM Leaderboard**: huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard
- **Ollama Library**: ollama.ai/library
- **LM Studio**: GUI for finding and running local models
