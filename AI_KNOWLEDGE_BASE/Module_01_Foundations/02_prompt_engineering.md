# Prompt Engineering

## What Is Prompt Engineering?

The practice of designing inputs (prompts) to get the best outputs from an LLM. It's both a science and an art — small wording changes can dramatically affect output quality.

---

## Core Techniques

### 1. Zero-Shot Prompting
Ask the model to do something without any examples.

```
Classify the sentiment of this review as Positive, Negative, or Neutral:
"The product arrived on time but the quality was poor."
```

### 2. Few-Shot Prompting
Provide examples before the actual task to guide the model.

```
Classify sentiment:
"Amazing product!" → Positive
"Worst purchase ever." → Negative
"It's okay I guess." → Neutral

"Delivery was fast but packaging was damaged." →
```

### 3. Chain-of-Thought (CoT)
Ask the model to reason step by step before giving an answer. Significantly improves performance on complex tasks.

```
Q: A train travels 60mph for 2 hours, then 80mph for 1 hour. What is the total distance?

Let's think step by step:
- First leg: 60 mph × 2 hours = 120 miles
- Second leg: 80 mph × 1 hour = 80 miles
- Total: 120 + 80 = 200 miles
```

Just adding "Let's think step by step" often activates CoT reasoning.

### 4. System Prompts
A special instruction block that defines the model's **persona, role, and rules**. Runs before the user message.

```
System: You are a senior Python developer. Answer only with code.
        Do not explain unless asked. Use Python 3.11+ syntax.

User: Write a function to flatten a nested list.
```

### 5. Role Prompting
Assign a role to get domain-specific responses.

```
You are an expert in cybersecurity. Review the following code for vulnerabilities...
```

### 6. Instruction Tuning Format
Be explicit and structured:
- State the task clearly
- Specify format of output (JSON, bullet list, table)
- Set constraints (max length, language, tone)

```
Summarize the following article in exactly 3 bullet points.
Each bullet point must be under 20 words. Use plain English.

Article: [...]
```

### 7. Delimiters
Use clear delimiters to separate instructions from content.

```
Translate the text inside triple backticks to French.

```Hello, how are you today?```
```

---

## Advanced Techniques

### ReAct (Reason + Act)
Model reasons about what to do, takes an action (tool call), observes the result, then reasons again. Foundation of AI agents.

### Self-Consistency
Run the same prompt multiple times, then take the majority answer. Improves accuracy on reasoning tasks.

### Prompt Chaining
Break complex tasks into a series of smaller prompts where the output of one feeds into the next.

```
Step 1: Extract key points from document
Step 2: Rank key points by importance
Step 3: Write a summary based on top 3 points
```

---

## Prompt Anti-Patterns (What NOT to Do)

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Vague instructions | Model guesses intent | Be explicit and specific |
| No output format | Inconsistent responses | Specify format (JSON, list, table) |
| Too long context | Key info gets lost | Put critical info at start/end |
| Negative instructions only | Model may still do it | Say what TO do, not just what NOT to do |
| Overloading one prompt | Low quality on all tasks | Use prompt chaining |

---

## Prompt Template Example (Production-Ready)

```python
SYSTEM_PROMPT = """
You are a helpful customer support assistant for Acme Corp.

Rules:
- Only answer questions about Acme Corp products
- If you don't know, say "I don't have that information"
- Never make up product specifications
- Always be polite and professional
- Respond in the same language as the user
"""

USER_PROMPT = """
Customer question: {question}

Relevant product info: {context}

Provide a helpful response:
"""
```

---

## Key Terms

| Term | Definition |
|------|-----------|
| Zero-shot | No examples provided |
| Few-shot | Examples provided in the prompt |
| CoT | Model shows reasoning steps |
| System prompt | Background instructions for the model |
| Temperature | Controls output randomness |
| Prompt injection | Malicious input that hijacks the prompt |

---

## Resources

- [Anthropic Prompt Engineering Guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)
- [Prompt Engineering Guide (dair-ai)](https://www.promptingguide.ai/)
