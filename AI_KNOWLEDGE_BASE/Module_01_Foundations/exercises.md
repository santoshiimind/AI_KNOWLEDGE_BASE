# Module 01 — Exercises

## Exercise 1: Token Counter
Build a function that takes any text and prints:
- Character count
- Word count
- Estimated token count (use the `~4 chars = 1 token` rule)
- The tokenized breakdown using `tiktoken`

```python
# Starter
import tiktoken

def analyze_text(text: str):
    # Your code here
    pass

# Test with these inputs
texts = [
    "Hello, world!",
    "The quick brown fox jumps over the lazy dog.",
    "def fibonacci(n): return n if n <= 1 else fibonacci(n-1) + fibonacci(n-2)"
]
```

**Goal**: Understand why tokenization matters for API costs.

---

## Exercise 2: Prompt Technique Comparison
Run the same question with 3 different prompt techniques and compare outputs:
1. Zero-shot
2. Few-shot (provide 3 examples)
3. Chain-of-thought

```python
import anthropic

client = anthropic.Anthropic()
question = "A store sells apples for $0.50 each and oranges for $0.75 each. If I buy 4 apples and 3 oranges, how much do I spend?"

def zero_shot(q): ...
def few_shot(q): ...
def chain_of_thought(q): ...

# Compare the results
```

**Goal**: See firsthand how prompt technique changes accuracy on math problems.

---

## Exercise 3: API Provider Switcher
Write a single `chat(message)` function that works with both Anthropic and OpenAI, switching based on a `PROVIDER` environment variable.

```python
import os

PROVIDER = os.environ.get("PROVIDER", "anthropic")  # or "openai"

def chat(message: str) -> str:
    if PROVIDER == "anthropic":
        # use Anthropic
        pass
    elif PROVIDER == "openai":
        # use OpenAI
        pass

# Test: run with PROVIDER=anthropic, then PROVIDER=openai
# Output should be the same format regardless of provider
```

**Goal**: Build provider-agnostic code from the start.

---

## Exercise 4: Temperature Explorer
Ask the same creative question 5 times with temperature 0, then 5 times with temperature 1.0. Count how many unique answers you get each time.

```python
def compare_temperatures(prompt: str):
    results_t0 = [call_with_temp(prompt, temp=0) for _ in range(5)]
    results_t1 = [call_with_temp(prompt, temp=1.0) for _ in range(5)]

    print(f"Temp=0: {len(set(results_t0))} unique responses out of 5")
    print(f"Temp=1: {len(set(results_t1))} unique responses out of 5")
```

**Goal**: Viscerally understand what temperature does.

---

## Exercise 5: System Prompt Tester
Build a simple CLI that lets you:
1. Enter a system prompt
2. Have a multi-turn conversation
3. Type `reset` to clear history
4. Type `newsystem` to change the system prompt

**Goal**: Internalize how system prompts shape model behavior.
