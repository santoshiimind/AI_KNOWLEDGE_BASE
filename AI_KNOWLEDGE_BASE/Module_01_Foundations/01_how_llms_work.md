# How LLMs Work

## Key Concepts

### Transformer Architecture
- LLMs are built on the **Transformer** architecture (introduced in "Attention Is All You Need", 2017)
- Core mechanism: **self-attention** — the model learns which parts of the input to focus on
- Consists of an encoder and/or decoder stack of layers

### Tokens
- LLMs don't read text as characters or words — they read **tokens**
- A token ≈ 4 characters or ~0.75 words on average
- Example: `"Hello, world!"` = 4 tokens
- Why it matters: API pricing, context limits, and performance are all measured in tokens
- Tool to visualize: https://platform.openai.com/tokenizer

### Embeddings
- Text is converted into **vectors** (arrays of numbers) called embeddings
- Similar meanings = similar vectors (close in vector space)
- Example: `"king" - "man" + "woman" ≈ "queen"`
- Used in: semantic search, RAG, classification, clustering

### Context Window
- The maximum number of tokens an LLM can process in one request (input + output)
- Examples (as of 2025):
  - GPT-4o: 128K tokens
  - Claude 3.5 Sonnet: 200K tokens
  - Gemini 1.5 Pro: 1M tokens
- Content outside the context window is "forgotten"

### How LLMs Generate Text
1. Input text is tokenized
2. Tokens are converted to embeddings
3. Transformer layers process the embeddings using attention
4. Model predicts the **next most likely token** (autoregressive)
5. This repeats until the output is complete
- The model is probabilistic — **temperature** controls randomness

### Temperature & Sampling
- **Temperature = 0**: deterministic, always picks most likely token
- **Temperature = 1**: balanced creativity
- **Temperature > 1**: more random/creative, less reliable
- **Top-p (nucleus sampling)**: another way to control output diversity

### Training vs Inference
- **Pre-training**: model learns language from massive text datasets
- **Fine-tuning**: model is further trained on domain-specific data
- **RLHF** (Reinforcement Learning from Human Feedback): aligns model to be helpful, harmless, honest
- **Inference**: using the trained model to generate responses

---

## Key Terms

| Term | Definition |
|------|-----------|
| Token | Smallest unit of text the model processes |
| Embedding | Numerical vector representation of text |
| Context Window | Max tokens in one request |
| Temperature | Controls output randomness |
| Inference | Running the model to get a response |
| RLHF | Training technique to align model behavior |
| Hallucination | Model confidently outputs incorrect information |

---

## Common Misconceptions

- LLMs do NOT "look up" information — they recall patterns from training
- They do NOT "understand" — they predict statistically likely next tokens
- Larger ≠ always better — smaller, fine-tuned models can outperform large general ones

---

## Resources

- [3Blue1Brown — But what is a GPT?](https://www.youtube.com/watch?v=wjZofJX0v4M)
- [Attention Is All You Need (paper)](https://arxiv.org/abs/1706.03762)
- [Andrej Karpathy — Let's build GPT from scratch](https://www.youtube.com/watch?v=kCc8FmEb1nY)
