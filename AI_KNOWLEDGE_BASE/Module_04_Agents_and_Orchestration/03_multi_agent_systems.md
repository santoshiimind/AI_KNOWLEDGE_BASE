# Multi-Agent Systems

## Why Multiple Agents?

- **Specialization**: Each agent focuses on what it does best
- **Parallelism**: Multiple agents work concurrently
- **Scale**: Break large tasks into independent subtasks
- **Quality**: Agents check each other's work

---

## Patterns

### 1. Orchestrator + Subagents

An orchestrator receives the task, delegates to specialist agents, and synthesizes results.

```
User → Orchestrator
           ├── Research Agent (web search)
           ├── Writer Agent (content creation)
           └── Editor Agent (review & improve)
         ↓
      Final Output
```

```python
import anthropic
import asyncio

client = anthropic.Anthropic()

def research_agent(topic: str) -> str:
    """Agent specialized in research."""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system="You are a research specialist. Find and summarize key facts.",
        messages=[{"role": "user", "content": f"Research: {topic}"}]
    )
    return response.content[0].text

def writer_agent(topic: str, research: str) -> str:
    """Agent specialized in writing."""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        system="You are a professional writer. Create clear, engaging content.",
        messages=[{
            "role": "user",
            "content": f"Write an article about '{topic}' using this research:\n{research}"
        }]
    )
    return response.content[0].text

def editor_agent(content: str) -> str:
    """Agent specialized in editing."""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        system="You are a professional editor. Improve clarity, fix errors, enhance flow.",
        messages=[{
            "role": "user",
            "content": f"Edit and improve this content:\n{content}"
        }]
    )
    return response.content[0].text

def orchestrator(task: str) -> str:
    """Orchestrates specialist agents to complete the task."""
    # Step 1: Research
    print("→ Research agent working...")
    research = research_agent(task)

    # Step 2: Write
    print("→ Writer agent working...")
    draft = writer_agent(task, research)

    # Step 3: Edit
    print("→ Editor agent working...")
    final = editor_agent(draft)

    return final

result = orchestrator("The impact of AI on software development")
print(result)
```

---

### 2. Parallel Agents

Run multiple agents concurrently for speed:

```python
import asyncio
import anthropic

async_client = anthropic.AsyncAnthropic()

async def specialized_agent(role: str, task: str) -> dict:
    response = await async_client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        system=f"You are a {role}. Give your expert perspective.",
        messages=[{"role": "user", "content": task}]
    )
    return {"role": role, "response": response.content[0].text}

async def parallel_analysis(topic: str) -> dict:
    agents = [
        ("technical expert", f"Analyze the technical aspects of: {topic}"),
        ("business analyst", f"Analyze the business impact of: {topic}"),
        ("risk assessor", f"Identify risks and challenges of: {topic}"),
        ("market researcher", f"Analyze market opportunities for: {topic}")
    ]

    # Run all agents in parallel
    tasks = [specialized_agent(role, task) for role, task in agents]
    results = await asyncio.gather(*tasks)

    return {r["role"]: r["response"] for r in results}

results = asyncio.run(parallel_analysis("Building AI-powered apps"))
for role, analysis in results.items():
    print(f"\n=== {role.upper()} ===")
    print(analysis)
```

---

### 3. Agent Pipeline (Assembly Line)

Output of each agent becomes input of the next:

```python
def pipeline(input_data: str, stages: list) -> str:
    """
    stages: list of (agent_name, system_prompt) tuples
    """
    current = input_data

    for stage_name, system_prompt in stages:
        print(f"→ Stage: {stage_name}")
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            system=system_prompt,
            messages=[{"role": "user", "content": current}]
        )
        current = response.content[0].text

    return current

stages = [
    ("Extractor", "Extract the key data points from this text. Return as bullet points."),
    ("Analyzer", "Analyze these data points. Identify patterns and insights."),
    ("Reporter", "Write an executive summary based on these insights. Max 200 words."),
]

result = pipeline(raw_document, stages)
```

---

### 4. Critic-Actor Pattern

One agent produces output, another critiques it:

```python
def critic_actor(task: str, iterations: int = 3) -> str:
    # Actor creates initial response
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{"role": "user", "content": task}]
    ).content[0].text

    for i in range(iterations):
        # Critic evaluates
        critique = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=512,
            system="You are a strict critic. Find flaws, gaps, and improvements needed. Be specific.",
            messages=[{
                "role": "user",
                "content": f"Task: {task}\n\nResponse to critique:\n{response}"
            }]
        ).content[0].text

        # Actor improves based on critique
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            messages=[{
                "role": "user",
                "content": f"Task: {task}\n\nPrevious response:\n{response}\n\nCritique:\n{critique}\n\nImproved response:"
            }]
        ).content[0].text

    return response
```

---

## Inter-Agent Communication

Agents communicate through structured messages:

```python
from dataclasses import dataclass
from typing import Any

@dataclass
class AgentMessage:
    sender: str
    receiver: str
    message_type: str  # "task", "result", "error", "request"
    content: Any
    metadata: dict = None

# Simple message queue
message_queue = []

def send_message(sender, receiver, msg_type, content):
    message_queue.append(AgentMessage(sender, receiver, msg_type, content))

def receive_messages(agent_name: str) -> list[AgentMessage]:
    messages = [m for m in message_queue if m.receiver == agent_name]
    for m in messages:
        message_queue.remove(m)
    return messages
```

---

## When to Use Multi-Agent

| Use Multi-Agent When | Use Single Agent When |
|---------------------|----------------------|
| Task has independent subtasks | Task is sequential |
| Need specialized expertise | General-purpose is fine |
| Quality needs peer review | One pass is sufficient |
| Time matters (parallelism) | Simplicity matters more |
| Task is too complex for one | Task fits in one context window |
