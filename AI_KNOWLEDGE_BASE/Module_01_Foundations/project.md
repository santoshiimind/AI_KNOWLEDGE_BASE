# Module 01 — Project: Prompt Lab CLI

## What You'll Build

A command-line tool for experimenting with prompts. Think of it as your personal prompt engineering workbench.

---

## Features to Implement

### Core (Required)
- [ ] Load a system prompt from a `.txt` file
- [ ] Interactive chat loop with conversation history
- [ ] Show token usage after every response
- [ ] Show estimated cost (input + output tokens × price)
- [ ] Commands: `reset`, `history`, `exit`

### Stretch Goals
- [ ] Save/load conversation history to JSON
- [ ] Support both Anthropic and OpenAI with a `--provider` flag
- [ ] `--temperature` flag
- [ ] Color-coded output (user vs assistant)
- [ ] `compare` command: run the last user message with 3 different temperatures and show all responses side-by-side

---

## Project Structure

```
prompt_lab/
├── main.py          # entry point
├── config.py        # model, provider, pricing config
├── chat.py          # LLM call logic
├── history.py       # conversation history management
└── prompts/
    ├── default.txt
    └── coder.txt
```

---

## Sample Usage

```bash
$ python main.py --system prompts/coder.txt --provider anthropic

System: [Loaded from prompts/coder.txt]
Model: claude-haiku-4-5-20251001 | Temp: 0.7

You: write a python decorator that times a function
Assistant: Here's a timing decorator...
[Tokens: 12 in / 87 out | Cost: $0.0003]

You: history
[Showing 2 messages...]

You: reset
[Conversation cleared]
```

---

## Hints

- Use `argparse` for CLI arguments
- Use `colorama` for colored terminal output: `pip install colorama`
- Pricing dict: `{"claude-haiku-4-5-20251001": {"input": 0.80, "output": 4.00}}` (per 1M tokens)
- Store history as `[{"role": "user"|"assistant", "content": "..."}]`

---

## What You'll Learn

- Full request/response cycle with an LLM API
- Token counting and cost awareness
- Conversation state management
- CLI tool design
