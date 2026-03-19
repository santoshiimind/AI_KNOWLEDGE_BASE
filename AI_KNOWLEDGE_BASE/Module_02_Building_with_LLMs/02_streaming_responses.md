# Streaming Responses

## Why Stream?

- Without streaming: user waits for the entire response (can be 5-30+ seconds)
- With streaming: tokens appear one by one, like typing — feels instant
- Essential for any chat UI or long-form generation

---

## Streaming — Anthropic

```python
import anthropic

client = anthropic.Anthropic()

with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a short story about a robot."}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

print()  # newline at end

# Get final message with usage stats
final_message = stream.get_final_message()
print(f"\nTokens: {final_message.usage.input_tokens} in, {final_message.usage.output_tokens} out")
```

## Streaming — OpenAI

```python
from openai import OpenAI

client = OpenAI()

stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Write a short story about a robot."}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content is not None:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

---

## Streaming in a Web App (FastAPI + Server-Sent Events)

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import anthropic

app = FastAPI()
client = anthropic.Anthropic()

@app.post("/chat")
async def chat(request: dict):
    user_message = request["message"]

    def generate():
        with client.messages.stream(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            messages=[{"role": "user", "content": user_message}]
        ) as stream:
            for text in stream.text_stream:
                # Server-Sent Events format
                yield f"data: {text}\n\n"
        yield "data: [DONE]\n\n"

    return StreamingResponse(
        generate(),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "X-Accel-Buffering": "no"
        }
    )
```

## Frontend (JavaScript) — Reading SSE Stream

```javascript
async function streamChat(message) {
    const response = await fetch('/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message })
    });

    const reader = response.body.getReader();
    const decoder = new TextDecoder();

    while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        const chunk = decoder.decode(value);
        const lines = chunk.split('\n');

        for (const line of lines) {
            if (line.startsWith('data: ')) {
                const text = line.slice(6);
                if (text === '[DONE]') return;
                document.getElementById('output').innerText += text;
            }
        }
    }
}
```

---

## Streaming with Event Hooks (Anthropic)

```python
import anthropic

client = anthropic.Anthropic()

class MyStreamHandler(anthropic.MessageStreamManager):
    def on_text(self, text):
        print(text, end="", flush=True)

    def on_message(self, message):
        print(f"\n\nDone! Total tokens: {message.usage.input_tokens + message.usage.output_tokens}")

with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=512,
    messages=[{"role": "user", "content": "Tell me a joke."}]
) as stream:
    stream.until_done()
```

---

## Key Points

- Always set `flush=True` when printing streamed tokens to terminal
- In web apps, use Server-Sent Events (SSE) or WebSockets
- Streaming doesn't reduce total tokens — it only affects when you receive them
- You can still get final usage stats after the stream completes
- Handle stream interruptions (user closes connection) gracefully
