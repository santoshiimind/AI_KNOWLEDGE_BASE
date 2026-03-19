# Function Calling & Tool Use

## What Is Tool Use?

Tool use (also called function calling) lets you give an LLM the ability to call your code.
The model decides **when** to call a tool and **what arguments** to pass — you execute it and return the result.

This is the foundation of AI agents.

---

## How It Works

```
1. You define tools (name, description, parameters)
2. User asks a question
3. Model decides if a tool is needed → returns a tool_call
4. You execute the function with the given args
5. You return the result to the model
6. Model generates final response using the tool result
```

---

## Tool Use — Anthropic

```python
import anthropic
import json

client = anthropic.Anthropic()

# Define your tools
tools = [
    {
        "name": "get_weather",
        "description": "Get the current weather for a specific location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City name, e.g. 'London' or 'New York'"
                },
                "unit": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "Temperature unit"
                }
            },
            "required": ["location"]
        }
    }
]

# Simulate the actual function
def get_weather(location: str, unit: str = "celsius") -> dict:
    # In real app, call a weather API here
    return {
        "location": location,
        "temperature": 22,
        "unit": unit,
        "condition": "Partly cloudy"
    }

def run_agent(user_message: str):
    messages = [{"role": "user", "content": user_message}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )

        # If model wants to use a tool
        if response.stop_reason == "tool_use":
            # Add assistant's response to history
            messages.append({"role": "assistant", "content": response.content})

            # Process each tool call
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"Calling tool: {block.name} with args: {block.input}")

                    # Execute the function
                    if block.name == "get_weather":
                        result = get_weather(**block.input)
                    else:
                        result = {"error": "Unknown tool"}

                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": json.dumps(result)
                    })

            # Return tool results to the model
            messages.append({"role": "user", "content": tool_results})

        else:
            # Model is done — extract final text response
            return response.content[0].text

answer = run_agent("What's the weather like in Tokyo?")
print(answer)
```

---

## Tool Use — OpenAI

```python
from openai import OpenAI
import json

client = OpenAI()

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
                },
                "required": ["location"]
            }
        }
    }
]

def get_weather(location, unit="celsius"):
    return {"location": location, "temperature": 22, "unit": unit}

messages = [{"role": "user", "content": "What's the weather in Paris?"}]

response = client.chat.completions.create(
    model="gpt-4o",
    tools=tools,
    messages=messages
)

if response.choices[0].finish_reason == "tool_calls":
    tool_call = response.choices[0].message.tool_calls[0]
    args = json.loads(tool_call.function.arguments)
    result = get_weather(**args)

    messages.append(response.choices[0].message)
    messages.append({
        "role": "tool",
        "tool_call_id": tool_call.id,
        "content": json.dumps(result)
    })

    final = client.chat.completions.create(model="gpt-4o", messages=messages)
    print(final.choices[0].message.content)
```

---

## Parallel Tool Calls

Models can call multiple tools in one response:

```python
# Example: "Get weather in London AND Paris"
# Model may return two tool_use blocks simultaneously

for block in response.content:
    if block.type == "tool_use":
        # Execute each in parallel (use asyncio.gather for real parallelism)
        result = execute_tool(block.name, block.input)
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": json.dumps(result)
        })
```

---

## Practical Tool Ideas

| Tool | Description |
|------|-------------|
| `search_web` | Search the internet for current info |
| `read_file` | Read a file from disk |
| `run_sql` | Execute a SQL query |
| `send_email` | Send an email |
| `get_stock_price` | Get current stock prices |
| `create_calendar_event` | Add an event to calendar |
| `call_api` | Make an HTTP request |
| `execute_code` | Run Python code in sandbox |

---

## Best Practices

1. Write clear, specific tool descriptions — the model reads these to decide when to call
2. Use strict parameter schemas with descriptions for each field
3. Always validate tool inputs before executing
4. Return structured results (JSON) from tools
5. Handle tool errors gracefully — return error info so model can respond appropriately
6. Set tool_choice to "auto" by default, "required" when you need a tool always called
