# Module 05 — Exercises

## Exercise 1: Persona Chatbot
Build a chatbot that plays a specific persona. The persona should be consistent across the conversation:

- Name: **Alex**, a senior software engineer at a startup
- Speaks casually, uses tech jargon, sometimes mentions coffee
- Refuses to discuss non-technical topics ("that's above my pay grade")
- Always gives code examples when explaining concepts

Test it by trying to break the persona (ask about cooking, politics, etc.)

```python
ALEX_SYSTEM_PROMPT = """
You are Alex, a senior software engineer...
"""

def chat_as_alex(history: list, user_msg: str) -> str:
    ...
```

---

## Exercise 2: Receipt Scanner
Build a vision-based receipt extractor. Take a photo of any receipt (or use a sample image from the internet) and extract:
- Store name
- Date
- Line items with prices
- Total
- Payment method (if visible)

Output it as a Python dataclass.

```python
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class ReceiptItem:
    name: str
    quantity: int
    price: float

@dataclass
class Receipt:
    store: str
    date: Optional[str]
    items: List[ReceiptItem]
    subtotal: float
    tax: float
    total: float

def scan_receipt(image_path: str) -> Receipt:
    ...
```

---

## Exercise 3: SQL Generator + Validator
Build a natural language to SQL converter with a twist: after generating the SQL, validate it by actually running it on a test SQLite database and return the results.

```python
import sqlite3

# Setup a test database
def create_test_db():
    conn = sqlite3.connect(":memory:")
    conn.executescript("""
        CREATE TABLE customers (id INTEGER, name TEXT, city TEXT, spend REAL);
        CREATE TABLE orders (id INTEGER, customer_id INTEGER, product TEXT, amount REAL, date TEXT);
        INSERT INTO customers VALUES (1, 'Alice', 'London', 1200.0);
        INSERT INTO customers VALUES (2, 'Bob', 'Paris', 850.0);
        -- add more data
    """)
    return conn

def text_to_sql_and_run(question: str, conn) -> dict:
    schema = get_schema(conn)
    sql = generate_sql(question, schema)

    try:
        cursor = conn.execute(sql)
        results = cursor.fetchall()
        return {"sql": sql, "results": results, "error": None}
    except Exception as e:
        return {"sql": sql, "results": None, "error": str(e)}

# Test questions:
questions = [
    "Who are the top 3 customers by total spend?",
    "How many orders were placed in each city?",
    "What is the average order amount?"
]
```

---

## Exercise 4: Image Batch Describer
Write a script that processes all images in a folder, generates a 1-sentence description for each, and saves the results to a CSV file.

```python
import os, csv
from pathlib import Path

def describe_image(image_path: str) -> str:
    # Use Claude vision
    ...

def batch_describe(folder: str, output_csv: str):
    images = [f for f in Path(folder).iterdir() if f.suffix in ('.jpg', '.png', '.jpeg')]

    with open(output_csv, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(["filename", "description"])

        for img_path in images:
            desc = describe_image(str(img_path))
            writer.writerow([img_path.name, desc])
            print(f"✓ {img_path.name}")
```

---

## Exercise 5: Meeting Transcription + Action Items
Record yourself speaking for 1-2 minutes about any topic (or use a sample audio file). Then:
1. Transcribe with Whisper
2. Extract action items using Claude
3. Extract key decisions
4. Format as a meeting summary email

```python
def process_meeting_recording(audio_path: str) -> dict:
    # Step 1: Transcribe
    transcript = transcribe(audio_path)

    # Step 2: Extract structure
    result = extract_meeting_data(transcript)
    # result = {
    #     "summary": "...",
    #     "action_items": [{"owner": "?", "task": "..."}],
    #     "decisions": ["..."],
    #     "follow_ups": ["..."]
    # }

    # Step 3: Format email
    email_body = format_as_email(result)
    return {"transcript": transcript, "structured": result, "email": email_body}
```
