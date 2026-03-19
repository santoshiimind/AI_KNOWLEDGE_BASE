# Module 02 — Project: Personal AI Assistant (Terminal)

## What You'll Build

A full-featured terminal AI assistant with streaming, tool use, and cost tracking. This is the foundation you'll extend in later modules.

---

## Features to Implement

### Core (Required)
- [ ] Streaming responses with live output
- [ ] Conversation history with sliding window
- [ ] 2 built-in tools: `get_current_time` and `calculate`
- [ ] Token + cost tracker (cumulative session total)
- [ ] Graceful error handling with user-friendly messages

### Stretch Goals
- [ ] `--export` flag to save conversation as Markdown
- [ ] Tool: `read_file(path)` — let the assistant read a file you point it to
- [ ] Tool: `web_search(query)` using DuckDuckGo or SerpAPI
- [ ] Switch models mid-conversation with `/model claude-haiku-4-5-20251001`
- [ ] `/tokens` command to show current context size

---

## Project Structure

```
ai_assistant/
├── main.py           # CLI entry point
├── assistant.py      # core assistant logic (tool loop)
├── tools.py          # tool definitions and implementations
├── memory.py         # conversation history + trimming
├── cost_tracker.py   # token + cost tracking
└── config.py         # model config, pricing
```

---

## Tool Definitions to Implement

```python
# tools.py

TOOLS = [
    {
        "name": "get_current_time",
        "description": "Get the current date and time",
        "input_schema": {"type": "object", "properties": {}, "required": []}
    },
    {
        "name": "calculate",
        "description": "Evaluate a mathematical expression safely",
        "input_schema": {
            "type": "object",
            "properties": {
                "expression": {"type": "string", "description": "Math expression, e.g. '(15 * 8) / 3'"}
            },
            "required": ["expression"]
        }
    }
]

def execute_tool(name: str, args: dict) -> str:
    if name == "get_current_time":
        from datetime import datetime
        return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    if name == "calculate":
        import ast
        # Use ast.literal_eval or a safe math parser, NOT eval()
        try:
            # safe eval using ast
            tree = ast.parse(args["expression"], mode='eval')
            # only allow math operations
            result = eval(compile(tree, '<string>', 'eval'))
            return str(result)
        except Exception as e:
            return f"Error: {e}"
```

---

## Sample Session

```
=== AI Assistant ===
Model: claude-sonnet-4-6 | Session cost: $0.0000

You: what time is it and what's 15% of 847?

[Using tool: get_current_time]
[Using tool: calculate(847 * 0.15)]