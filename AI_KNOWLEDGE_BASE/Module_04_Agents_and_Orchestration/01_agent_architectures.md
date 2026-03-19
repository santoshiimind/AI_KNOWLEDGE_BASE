# Agent Architectures

## ReAct (Reason + Act)

The most common agent pattern. The model alternates between:
- **Thought**: reasoning about what to do
- **Action**: calling a tool
- **Observation**: receiving the tool result

```
User: What is the weather in Tokyo and should I bring an umbrella?

Thought: I need to find the current weather in Tokyo.
Action: get_weather(location="Tokyo")
Observation: {"temp": 18, "condition": "rainy", "humidity": 85}

Thought: It's rainy in Tokyo. The user should bring an umbrella.
Action: FINAL ANSWER
Answer: It's currently rainy in Tokyo (18°C). Yes, you should definitely bring an umbrella!
```

### ReAct Implementation

```python
import anthropic
import json

client = anthropic.Anthropic()

# Tool definitions
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather for a location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "City name"}
            },
            "required": ["location"]
        }
    },
    {
        "name": "search_web",
        "description": "Search the web for current information",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"}
            },
            "required": ["query"]
        }
    }
]

# Tool implementations
def get_weather(location: str) -> dict:
    # Replace with real API call
    return {"location": location, "temp": 18, "condition": "rainy"}

def search_web(query: str) -> str:
    # Replace with real search API
    return f"Search results for '{query}': ..."

def execute_tool(name: str, args: dict) -> str:
    if name == "get_weather":
        result = get_weather(**args)
    elif name == "search_web":
        result = search_web(**args)
    else:
        result = {"error": f"Unknown tool: {name}"}
    return json.dumps(result)

# Agent loop
def run_agent(user_message: str, max_iterations: int = 10) -> str:
    messages = [{"role": "user", "content": user_message}]

    for iteration in range(max_iterations):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )

        # Model finished — return answer
        if response.stop_reason == "end_turn":
            for block in response.content:
                if hasattr(block, "text"):
                    return block.text
            return "Task completed."

        # Model wants to use a tool
        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})

            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"[Tool Call] {block.name}({block.input})")
                    result = execute_tool(block.name, block.input)
                    print(f"[Tool Result] {result[:100]}...")

                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })

            messages.append({"role": "user", "content": tool_results})

    return "Max iterations reached without a final answer."

answer = run_agent("What's the weather in Tokyo and should I bring an umbrella?")
print(answer)
```

---

## Plan-and-Execute Architecture

For complex tasks that require upfront planning:

```
1. PLAN: Break the task into subtasks
2. EXECUTE: Execute each subtask (possibly with separate agents)
3. SYNTHESIZE: Combine results into final answer
```

```python
def plan_and_execute(task: str) -> str:
    # Step 1: Create a plan
    plan_response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        messages=[{
            "role": "user",
            "content": f"""Break this task into 3-5 concrete steps. Return as JSON array of strings.
Task: {task}"""
        }]
    )
    steps = json.loads(plan_response.content[0].text)
    print(f"Plan: {steps}")

    # Step 2: Execute each step
    results = []
    for step in steps:
        result = run_agent(step)
        results.append({"step": step, "result": result})

    # Step 3: Synthesize
    synthesis = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": f"""Original task: {task}

Steps completed:
{json.dumps(results, indent=2)}

Synthesize all results into a final comprehensive answer."""
        }]
    )
    return synthesis.content[0].text
```

---

## Reflection Architecture

The agent critiques and improves its own output:

```python
def reflect_and_improve(task: str, initial_response: str) -> str:
    critique = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        messages=[{
            "role": "user",
            "content": f"""Task: {task}

Response: {initial_response}

Critique this response. What's missing, incorrect, or could be improved?
Be specific and actionable."""
        }]
    ).content[0].text

    improved = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": f"""Task: {task}

Original response: {initial_response}

Critique: {critique}

Now provide an improved response that addresses all the critique points."""
        }]
    ).content[0].text

    return improved
```

---

## Guardrails & Stopping Conditions

Always implement safeguards for agent loops:

```python
class AgentConfig:
    max_iterations: int = 10       # prevent infinite loops
    max_cost_usd: float = 0.50     # cost budget
    timeout_seconds: int = 60      # time limit
    allowed_tools: list = None     # restrict which tools can be used

def run_safe_agent(task: str, config: AgentConfig) -> str:
    import time
    start_time = time.time()
    total_tokens = 0

    for i in range(config.max_iterations):
        if time.time() - start_time > config.timeout_seconds:
            return "Timeout: agent took too long"

        # ... agent step ...

        total_tokens += response.usage.input_tokens + response.usage.output_tokens
        estimated_cost = total_tokens / 1_000_000 * 18  # rough estimate
        if estimated_cost > config.max_cost_usd:
            return "Budget exceeded"
```

---

## Key Concepts

| Concept | Description |
|---------|-------------|
| Agent loop | Reason → Act → Observe cycle |
| ReAct | Most common agent pattern |
| Tool calling | How agents interact with the world |
| Stop condition | When the agent decides it's done |
| Max iterations | Safety limit on agent loops |
| Scratchpad | The agent's internal reasoning |
