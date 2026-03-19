# Module 05 — Project: Full-Stack AI Web App

## What You'll Build

A web-based document Q&A chatbot. Users can upload documents, ask questions about them via a chat interface, and get streaming answers with source citations.

This brings together RAG + chatbot + streaming + document processing in one app.

---

## Features to Implement

### Core (Required)
- [ ] FastAPI backend with REST + WebSocket endpoints
- [ ] Simple HTML/JS frontend (no framework required)
- [ ] Upload documents (PDF/TXT) via browser
- [ ] Chat interface with streaming responses
- [ ] Source citations shown below each answer
- [ ] Conversation history per session
- [ ] List uploaded documents

### Stretch Goals
- [ ] Drag-and-drop file upload
- [ ] Delete documents
- [ ] Multiple chat sessions (sidebar)
- [ ] Copy answer to clipboard button
- [ ] Export chat as PDF
- [ ] Show "thinking..." animation while waiting for first token

---

## Project Structure

```
ai_webapp/
├── backend/
│   ├── main.py          # FastAPI app
│   ├── rag.py           # indexing + retrieval
│   ├── chat.py          # LLM call + streaming
│   └── store.py         # ChromaDB wrapper
├── frontend/
│   ├── index.html       # single-page app
│   ├── style.css
│   └── app.js           # fetch + WebSocket + UI logic
└── uploads/             # uploaded documents
```

---

## API Design

```
POST   /upload              → upload + index a document
GET    /documents           → list all indexed documents
DELETE /documents/{name}    → remove a document
POST   /chat                → send message, get streaming response (SSE)
GET    /health              → health check
```

---

## Backend Skeleton

```python
# backend/main.py
from fastapi import FastAPI, UploadFile, File
from fastapi.responses import StreamingResponse
from fastapi.staticfiles import StaticFiles
from fastapi.middleware.cors import CORSMiddleware
import json, os

app = FastAPI()
app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_methods=["*"], allow_headers=["*"])
app.mount("/", StaticFiles(directory="../frontend", html=True), name="frontend")

@app.post("/upload")
async def upload_document(file: UploadFile = File(...)):
    content = await file.read()
    path = f"uploads/{file.filename}"
    with open(path, "wb") as f:
        f.write(content)
    chunks_added = index_document(path)
    return {"filename": file.filename, "chunks": chunks_added}

@app.post("/chat")
async def chat(request: dict):
    question = request["message"]
    history = request.get("history", [])

    def generate():
        chunks = retrieve(question)
        for token in stream_answer(question, chunks, history):
            yield f"data: {json.dumps({'token': token})}\n\n"

        sources = list(set(c["source"] for c in chunks))
        yield f"data: {json.dumps({'done': True, 'sources': sources})}\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

---

## Frontend Skeleton

```html
<!-- frontend/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>AI Document Chat</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="sidebar">
        <h2>Documents</h2>
        <input type="file" id="fileInput" accept=".pdf,.txt,.md">
        <button onclick="uploadFile()">Upload</button>
        <ul id="docList"></ul>
    </div>

    <div id="chat">
        <div id="messages"></div>
        <div id="input-area">
            <input id="userInput" placeholder="Ask a question..." />
            <button onclick="sendMessage()">Send</button>
        </div>
    </div>

    <script src="app.js"></script>
</body>
</html>
```

```javascript
// frontend/app.js
let history = [];

async function sendMessage() {
    const input = document.getElementById('userInput');
    const message = input.value.trim();
    if (!message) return;
    input.value = '';

    appendMessage('user', message);

    const response = await fetch('/chat', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({ message, history })
    });

    const msgEl = appendMessage('assistant', '');
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let fullText = '';

    while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        const lines = decoder.decode(value).split('\n');
        for (const line of lines) {
            if (!line.startsWith('data: ')) continue;
            const data = JSON.parse(line.slice(6));

            if (data.token) {
                fullText += data.token;
                msgEl.textContent = fullText;
            }
            if (data.done && data.sources?.length) {
                const srcEl = document.createElement('div');
                srcEl.className = 'sources';
                srcEl.textContent = 'Sources: ' + data.sources.join(', ');
                msgEl.after(srcEl);
            }
        }
    }

    history.push({role: 'user', content: message});
    history.push({role: 'assistant', content: fullText});
}
```

---

## What You'll Learn

- FastAPI backend with file uploads and streaming
- SSE (Server-Sent Events) for real-time streaming to browser
- Combining RAG + chat in a real application
- Basic frontend that consumes a streaming AI API
- Full-stack thinking: how all the pieces connect
