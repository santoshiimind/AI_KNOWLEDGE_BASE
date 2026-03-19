# Chatbots & Conversational Apps

## Core Chatbot Architecture

```
User Input
    ↓
[Message Validation]
    ↓
[Load Conversation History]
    ↓
[Build Messages Array]
    ↓
[LLM API Call]
    ↓
[Stream Response to User]
    ↓
[Save to History]
```

---

## Minimal Chatbot (CLI)

```python
import anthropic

client = anthropic.Anthropic()

def run_chatbot():
    history = []
    system = "You are a helpful assistant. Be concise and friendly."

    print("Chat started. Type 'quit' to exit.\n")

    while True:
        user_input = input("You: ").strip()
        if user_input.lower() in ("quit", "exit"):
            break
        if not user_input:
            continue

        history.append({"role": "user", "content": user_input})

        print("Assistant: ", end="", flush=True)

        full_response = ""
        with client.messages.stream(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            system=system,
            messages=history
        ) as stream:
            for text in stream.text_stream:
                print(text, end="", flush=True)
                full_response += text

        print()  # newline
        history.append({"role": "assistant", "content": full_response})

run_chatbot()
```

---

## Production Chatbot (FastAPI + WebSocket)

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.middleware.cors import CORSMiddleware
import anthropic
import json

app = FastAPI()
app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_methods=["*"], allow_headers=["*"])

client = anthropic.Anthropic()

# In production, store sessions in Redis/DB
sessions = {}

@app.websocket("/ws/{session_id}")
async def websocket_chat(websocket: WebSocket, session_id: str):
    await websocket.accept()

    if session_id not in sessions:
        sessions[session_id] = []

    try:
        while True:
            data = await websocket.receive_text()
            message = json.loads(data)
            user_text = message["content"]

            sessions[session_id].append({
                "role": "user",
                "content": user_text
            })

            # Stream response back via WebSocket
            full_response = ""

            with client.messages.stream(
                model="claude-sonnet-4-6",
                max_tokens=1024,
                system="You are a helpful assistant.",
                messages=sessions[session_id]
            ) as stream:
                for text in stream.text_stream:
                    full_response += text
                    await websocket.send_json({
                        "type": "token",
                        "content": text
                    })

            # Signal completion
            await websocket.send_json({"type": "done"})

            sessions[session_id].append({
                "role": "assistant",
                "content": full_response
            })

    except WebSocketDisconnect:
        pass
```

---

## Adding Personality & Guardrails

```python
SYSTEM_PROMPT = """You are Aria, a customer support assistant for TechCorp.

Personality:
- Friendly and professional
- Empathetic when customers are frustrated
- Concise — keep responses under 3 paragraphs

Capabilities:
- Answer questions about TechCorp products
- Help troubleshoot common issues
- Process refund requests (use the submit_refund tool)

Limitations:
- Do NOT discuss competitors
- Do NOT make promises about future features
- Do NOT share internal company information
- If you can't help, offer to transfer to a human agent

Always:
- Start by acknowledging the customer's concern
- End with "Is there anything else I can help you with?"
"""
```

---

## Chatbot with Persona Switching

```python
PERSONAS = {
    "friendly": "You are warm, enthusiastic, and use casual language.",
    "professional": "You are formal, precise, and business-like.",
    "teacher": "You explain things step-by-step as if teaching a beginner.",
    "expert": "You give expert-level answers with technical depth."
}

def chat_with_persona(messages: list, persona: str = "friendly") -> str:
    system = PERSONAS.get(persona, PERSONAS["friendly"])
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=system,
        messages=messages
    )
    return response.content[0].text
```

---

## Input Validation & Safety

```python
import re

MAX_MESSAGE_LENGTH = 2000
BLOCKED_PATTERNS = [
    r"ignore (all |previous |above )?instructions",
    r"you are now",
    r"pretend you are",
    r"jailbreak",
]

def validate_input(user_input: str) -> tuple[bool, str]:
    if not user_input.strip():
        return False, "Empty message"

    if len(user_input) > MAX_MESSAGE_LENGTH:
        return False, f"Message too long (max {MAX_MESSAGE_LENGTH} chars)"

    for pattern in BLOCKED_PATTERNS:
        if re.search(pattern, user_input.lower()):
            return False, "Message contains prohibited content"

    return True, ""

# In your chat handler:
valid, reason = validate_input(user_input)
if not valid:
    return f"I can't process that message: {reason}"
```

---

## Key Production Concerns

| Concern | Solution |
|---------|---------|
| Session persistence | Redis or DB for history |
| Rate limiting | Per-user token bucket |
| Spam/abuse | Input validation + moderation API |
| Context overflow | Sliding window + summarization |
| Latency | Streaming + caching frequent queries |
| Cost control | Budget limits per user/session |
