# Fine-Tuning Basics

## Fine-Tuning vs Prompting vs RAG

| Approach | Best For | Cost | Effort |
|----------|---------|------|--------|
| **Prompting** | General tasks, flexible | Low | Low |
| **RAG** | Private/recent knowledge | Medium | Medium |
| **Fine-tuning** | Consistent style/format, specialized domain | High | High |

**Rule of thumb**: Try prompting first → then RAG → fine-tune only if necessary.

---

## When to Fine-Tune

Fine-tuning is worth it when:
- You need very consistent output format/style
- You have 100s-1000s of domain-specific examples
- The base model consistently fails even with good prompts
- You need faster/cheaper inference (smaller fine-tuned model can beat larger base)
- Prompt engineering has hit its limit

**Don't fine-tune for:**
- Giving the model new knowledge (use RAG instead)
- One-time tasks
- Tasks where prompting already works well

---

## OpenAI Fine-Tuning

### 1. Prepare Training Data

```python
import json

# Training data format: JSONL
# Each line is one training example
training_examples = [
    {
        "messages": [
            {"role": "system", "content": "You are a customer support agent for AcmeCorp."},
            {"role": "user", "content": "My order hasn't arrived"},
            {"role": "assistant", "content": "I apologize for the delay. Order #12345 shipped 3 days ago via FedEx. Tracking: 1234567890. Expected delivery: tomorrow by 8pm. Would you like me to contact FedEx?"}
        ]
    },
    # ... 50-1000+ more examples
]

# Save as JSONL
with open("training_data.jsonl", "w") as f:
    for example in training_examples:
        f.write(json.dumps(example) + "\n")
```

### 2. Upload & Train

```python
from openai import OpenAI

client = OpenAI()

# Upload training file
with open("training_data.jsonl", "rb") as f:
    file = client.files.create(file=f, purpose="fine-tune")

print(f"File ID: {file.id}")

# Start fine-tuning job
job = client.fine_tuning.jobs.create(
    training_file=file.id,
    model="gpt-4o-mini",  # base model to fine-tune
    hyperparameters={
        "n_epochs": 3,          # number of training passes
        "batch_size": "auto",
        "learning_rate_multiplier": "auto"
    }
)

print(f"Job ID: {job.id}")
print(f"Status: {job.status}")
```

### 3. Monitor & Use

```python
# Check status
job = client.fine_tuning.jobs.retrieve("ftjob-abc123")
print(f"Status: {job.status}")
print(f"Model: {job.fine_tuned_model}")

# List events
events = client.fine_tuning.jobs.list_events("ftjob-abc123")
for event in events.data:
    print(event.message)

# Use the fine-tuned model (exactly like normal API)
response = client.chat.completions.create(
    model=job.fine_tuned_model,  # e.g., "ft:gpt-4o-mini:your-org:custom-name:abc123"
    messages=[{"role": "user", "content": "My order hasn't arrived"}]
)
```

---

## Fine-Tuning Open Source Models (Local)

For Llama, Mistral, Phi, etc. — use Hugging Face + PEFT:

```bash
pip install transformers peft datasets accelerate bitsandbytes
```

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, TrainingArguments
from peft import LoraConfig, get_peft_model
from datasets import Dataset

# Load base model with 4-bit quantization (saves GPU memory)
from transformers import BitsAndBytesConfig
import torch

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16,
)

model_name = "meta-llama/Llama-3.1-8B"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config,
    device_map="auto"
)

# LoRA configuration (Parameter-Efficient Fine-Tuning)
lora_config = LoraConfig(
    r=16,           # rank — higher = more capacity, more memory
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],  # which layers to adapt
    lora_dropout=0.1,
    bias="none",
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# Typically only 0.1-1% of parameters are trainable with LoRA!
```

---

## Key Fine-Tuning Concepts

| Concept | Description |
|---------|-------------|
| **Full fine-tuning** | Update all model weights (expensive, needs many GPUs) |
| **LoRA** | Update only small adapter matrices (efficient, popular) |
| **QLoRA** | LoRA on quantized model (runs on single consumer GPU) |
| **PEFT** | Parameter-Efficient Fine-Tuning (umbrella term) |
| **Epochs** | How many times the model sees all training data |
| **Overfitting** | Model memorizes training data instead of generalizing |
| **Validation loss** | Metric to detect overfitting during training |

---

## Data Quality Guidelines

- **Quantity**: 50 minimum, 500+ recommended, 10K+ for complex tasks
- **Quality > Quantity**: 100 perfect examples > 1000 mediocre ones
- **Diversity**: Cover edge cases, not just happy paths
- **Consistency**: Same format, same style across all examples
- **No hallucinations**: Every example must be factually correct

---

## Fine-Tuning Checklist

```
Before fine-tuning:
□ Exhausted prompt engineering options?
□ Tried RAG if the issue is knowledge-based?
□ Have 100+ quality examples?
□ Have validation set (20% holdout)?
□ Defined success metric (how will you measure if it worked)?

During fine-tuning:
□ Monitor training/validation loss (should both decrease)
□ Check for overfitting (train loss drops, val loss rises)
□ Run evals at each checkpoint

After fine-tuning:
□ A/B test vs base model on eval set
□ Test on out-of-distribution examples
□ Check for regression on general tasks
□ Document the training setup for reproducibility
```
