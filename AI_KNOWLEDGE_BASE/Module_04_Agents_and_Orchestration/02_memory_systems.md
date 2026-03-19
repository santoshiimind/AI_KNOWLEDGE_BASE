# Memory Systems

## Types of Agent Memory

| Type | What it stores | Duration | Where |
|------|---------------|----------|-------|
| **In-context** | Current conversation | Single session | Messages array |
| **External (short-term)** | Session state | Hours/days | Redis, DB |
| **External (long-term)** | User preferences, facts | Permanent | Vector DB + DB |
| **Episodic** | Past interactions summary | Permanent | DB |

---

## 1. In-Context Memory (Conversation History)

The simplest form — the messages array IS the memory:

```python
class ConversationMemory:
    def __init__(self, max_messages: int = 20):
        self.messages = []
        self.max_messages = max_messages

    def add_user(self, content: str):
        self.messages.append({"role": "user", "content": content})
        self._trim()

    def add_assistant(self, content: str):
        self.messages.append({"role": "assistant", "content": content})
        self._trim()

    def _trim(self):
        # Keep only the most recent messages
        if len(self.messages) > self.max_messages:
            self.messages = self.messages[-self.max_messages:]

    def get_messages(self) -> list:
        return self.messages.copy()
```

**Limitation**: Lost when session ends, expensive for long conversations.

---

## 2. Summarization Memory

Compress old messages into a summary:

```python
import anthropic

client = anthropic.Anthropic()

class SummaryMemory:
    def __init__(self, keep_recent: int = 6):
        self.summary = ""
        self.recent_messages = []
        self.keep_recent = keep_recent

    def add(self, role: str, content: str):
        self.recent_messages.append({"role": role, "content": content})

        if len(self.recent_messages) > self.keep_recent * 2:
            self._compress()

    def _compress(self):
        old = self.recent_messages[:-self.keep_recent]
        self.recent_messages = self.recent_messages[-self.keep_recent:]

        history_text = "\n".join([f"{m['role']}: {m['content']}" for m in old])
        prompt = f"Previous summary: {self.summary}\n\nNew exchanges:\n{history_text}\n\nUpdate the summary:"

        response = client.messages.create(
            model="claude-haiku-4-5-20251001",
            max_tokens=300,
            messages=[{"role": "user", "content": prompt}]
        )
        self.summary = response.content[0].text

    def get_messages(self) -> list:
        messages = []
        if self.summary:
            messages.append({
                "role": "user",
                "content": f"[Conversation summary so far: {self.summary}]"
            })
            messages.append({"role": "assistant", "content": "Understood."})
        messages.extend(self.recent_messages)
        return messages
```

---

## 3. Semantic Memory (Long-Term, Vector-Based)

Store and retrieve facts based on semantic similarity:

```python
import chromadb
from chromadb.utils import embedding_functions
import anthropic

client = anthropic.Anthropic()
chroma = chromadb.PersistentClient(path="./agent_memory")

openai_ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key="your-key",
    model_name="text-embedding-3-small"
)

memory_store = chroma.get_or_create_collection(
    name="agent_memory",
    embedding_function=openai_ef
)

def save_memory(user_id: str, fact: str):
    """Save an important fact to long-term memory."""
    memory_store.add(
        documents=[fact],
        ids=[f"{user_id}_{hash(fact)}"],
        metadatas=[{"user_id": user_id, "fact": fact}]
    )

def recall_memories(user_id: str, query: str, top_k: int = 3) -> list[str]:
    """Retrieve relevant memories for the current context."""
    results = memory_store.query(
        query_texts=[query],
        n_results=top_k,
        where={"user_id": user_id}  # filter by user
    )
    return results["documents"][0] if results["documents"] else []

def extract_and_save_memories(user_id: str, conversation: str):
    """Use LLM to extract important facts worth remembering."""
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=300,
        messages=[{
            "role": "user",
            "content": f"""Extract 1-3 important facts from this conversation worth remembering long-term.
Only extract facts about the user's preferences, goals, or important information.
Return as a JSON array of strings, or empty array if nothing important.

Conversation:
{conversation}"""
        }]
    )
    import json
    facts = json.loads(response.content[0].text)
    for fact in facts:
        save_memory(user_id, fact)


# Usage in agent
def chat_with_memory(user_id: str, message: str, history: list) -> str:
    # Recall relevant long-term memories
    memories = recall_memories(user_id, message)

    memory_context = ""
    if memories:
        memory_context = "What I know about you:\n" + "\n".join(f"- {m}" for m in memories)

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=f"You are a helpful personal assistant.\n\n{memory_context}",
        messages=history + [{"role": "user", "content": message}]
    )

    return response.content[0].text
```

---

## 4. Episodic Memory (Past Interactions)

Store summaries of past sessions in a database:

```python
import sqlite3
from datetime import datetime

def setup_episodic_db():
    conn = sqlite3.connect("agent_episodes.db")
    conn.execute("""
        CREATE TABLE IF NOT EXISTS episodes (
            id INTEGER PRIMARY KEY,
            user_id TEXT,
            session_id TEXT,
            summary TEXT,
            timestamp TEXT
        )
    """)
    conn.commit()
    return conn

def save_episode(conn, user_id: str, session_id: str, conversation: list):
    # Summarize the session
    conv_text = "\n".join([f"{m['role']}: {m['content']}" for m in conversation])
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=200,
        messages=[{
            "role": "user",
            "content": f"Summarize this conversation in 2-3 sentences:\n{conv_text}"
        }]
    )
    summary = response.content[0].text

    conn.execute(
        "INSERT INTO episodes (user_id, session_id, summary, timestamp) VALUES (?, ?, ?, ?)",
        (user_id, session_id, summary, datetime.now().isoformat())
    )
    conn.commit()

def get_recent_episodes(conn, user_id: str, limit: int = 5) -> list[str]:
    cursor = conn.execute(
        "SELECT summary, timestamp FROM episodes WHERE user_id = ? ORDER BY timestamp DESC LIMIT ?",
        (user_id, limit)
    )
    return [f"[{row[1][:10]}] {row[0]}" for row in cursor.fetchall()]
```

---

## Memory Architecture Decision

```
Simple chatbot          → In-context memory only
Multi-session assistant → In-context + Episodic (DB)
Personal AI assistant   → All four types
Knowledge-heavy agent   → Semantic memory (RAG) dominant
```
