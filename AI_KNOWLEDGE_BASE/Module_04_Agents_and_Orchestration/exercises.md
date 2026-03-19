# Module 04 — Exercises

## Exercise 1: Build a Calculator Agent
Create an agent that can handle complex math word problems by breaking them into steps and using a `calculate` tool. The agent should show its reasoning before each tool call.

```python
# Tools: calculate(expression), get_unit_conversion(from_unit, to_unit, value)
# Test problems:
problems = [
    "If a car travels at 60mph for 2.5 hours, then at 80mph for 1.5 hours, what is the total distance in km?",
    "A rectangle has a perimeter of 48cm. If the length is twice the width, what is the area?",
    "If I invest $1000 at 5% annual interest compounded monthly, how much will I have after 3 years?"
]
```

**Goal**: Build and debug the full ReAct loop.

---

## Exercise 2: Agent with Web Search
Extend your agent with a simulated web search tool. Since you don't have a real search API, simulate it with a dict of pre-written "search results".

```python
FAKE_SEARCH_DB = {
    "python latest version": "Python 3.13 was released in October 2024...",
    "anthropic claude models": "Anthropic offers Claude Haiku, Sonnet, and Opus...",
    "openai gpt models": "OpenAI offers GPT-4o, GPT-4o-mini, and o1...",
    # add 10 more
}

def search_web(query: str) -> str:
    # Find best matching key in FAKE_SEARCH_DB
    # Fallback: "No results found"
    pass
```

Then ask: "Compare the latest Claude and GPT models and give me a recommendation for building a chatbot."

**Goal**: Practice multi-step reasoning with information retrieval.

---

## Exercise 3: Memory Agent
Build a simple agent that remembers facts about the user across a conversation:

1. When the user mentions a personal fact ("I live in London", "I'm a Python dev"), the agent calls `save_memory(fact)`
2. Before every response, it calls `recall_memories(topic)` to check for relevant memories
3. It should proactively use those memories ("Since you're in London, the time zone is...")

```python
memory_store = []  # in-memory for now

tools = [
    {"name": "save_memory", ...},
    {"name": "recall_memories", ...}
]
```

---

## Exercise 4: Multi-Agent Pipeline
Build a 3-stage pipeline for writing a blog post:
1. **Researcher Agent** → given a topic, outputs 5 key facts with sources
2. **Outline Agent** → given the facts, creates a 5-section outline
3. **Writer Agent** → given the outline, writes the full post

Each agent should only know about its own task. Pass output from one to the next.

```python
def research(topic: str) -> str: ...
def create_outline(research: str) -> str: ...
def write_post(outline: str) -> str: ...

# Chain them
result = write_post(create_outline(research("The future of AI in healthcare")))
```

---

## Exercise 5: Agent Safety — Guardrailed Agent
Take your calculator agent from Exercise 1 and add:
- Max 5 tool calls (raise an error if exceeded)
- Tool call logging (print every call with timestamp)
- Budget guard: stop if estimated cost exceeds $0.01
- Disallowed tools list (try adding `calculate` to the banned list and see what happens)

```python
class GuardrailedAgent:
    def __init__(self, max_tool_calls=5, max_cost_usd=0.01, banned_tools=None):
        ...

    def run(self, task: str) -> str:
        ...
```
