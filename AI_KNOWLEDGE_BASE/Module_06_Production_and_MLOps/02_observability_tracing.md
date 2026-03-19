# Observability & Tracing

## What to Monitor in AI Apps

- **Inputs**: what users are asking (detect misuse, spot trends)
- **Outputs**: what the model responds (quality, errors)
- **Latency**: how long each step takes
- **Token usage**: cost tracking
- **Errors**: API failures, parse errors, tool failures

---

## Basic Logging Wrapper

```python
import anthropic
import time
import logging
import json
from datetime import datetime

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("llm_calls")

client = anthropic.Anthropic()

def tracked_call(messages: list, system: str = None, **kwargs) -> str:
    start_time = time.time()
    request_id = f"req_{int(start_time * 1000)}"

    log_entry = {
        "request_id": request_id,
        "timestamp": datetime.utcnow().isoformat(),
        "model": kwargs.get("model", "claude-sonnet-4-6"),
        "input_messages": len(messages),
        "user_message": messages[-1]["content"][:200] if messages else ""
    }

    try:
        response = client.messages.create(
            system=system or "",
            messages=messages,
            **{"model": "claude-sonnet-4-6", "max_tokens": 1024, **kwargs}
        )

        duration_ms = (time.time() - start_time) * 1000

        log_entry.update({
            "status": "success",
            "duration_ms": round(duration_ms),
            "input_tokens": response.usage.input_tokens,
            "output_tokens": response.usage.output_tokens,
            "output_preview": response.content[0].text[:200]
        })

        logger.info(json.dumps(log_entry))
        return response.content[0].text

    except Exception as e:
        log_entry.update({
            "status": "error",
            "error": str(e),
            "duration_ms": round((time.time() - start_time) * 1000)
        })
        logger.error(json.dumps(log_entry))
        raise
```

---

## LangSmith (LangChain Tracing)

```bash
pip install langsmith
```

```python
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "your-langsmith-key"
os.environ["LANGCHAIN_PROJECT"] = "my-ai-app"

# All LangChain calls are now automatically traced
from langchain_anthropic import ChatAnthropic
llm = ChatAnthropic(model="claude-sonnet-4-6")

# Every call appears in the LangSmith dashboard with:
# - Full input/output
# - Token usage
# - Latency
# - Error traces
```

---

## Custom Tracing with OpenTelemetry

```bash
pip install opentelemetry-sdk opentelemetry-exporter-otlp
```

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

# Setup
provider = TracerProvider()
exporter = OTLPSpanExporter(endpoint="http://localhost:4317")
provider.add_span_processor(BatchSpanProcessor(exporter))
trace.set_tracer_provider(provider)

tracer = trace.get_tracer("llm-app")

def traced_rag_query(question: str) -> str:
    with tracer.start_as_current_span("rag_query") as span:
        span.set_attribute("question", question)

        # Retrieval step
        with tracer.start_as_current_span("retrieval"):
            chunks = retrieve(question)
            span.set_attribute("chunks_retrieved", len(chunks))

        # Generation step
        with tracer.start_as_current_span("generation") as gen_span:
            answer = generate_answer(question, chunks)
            gen_span.set_attribute("answer_length", len(answer))

        return answer
```

---

## Metrics Dashboard (Prometheus + Grafana)

```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Metrics
llm_requests_total = Counter("llm_requests_total", "Total LLM API calls", ["model", "status"])
llm_latency = Histogram("llm_latency_seconds", "LLM call latency", ["model"])
llm_tokens = Counter("llm_tokens_total", "Total tokens used", ["model", "type"])
active_sessions = Gauge("active_sessions", "Currently active chat sessions")

# Start metrics server on port 8001
start_http_server(8001)

def instrumented_call(messages, model="claude-sonnet-4-6"):
    start = time.time()
    try:
        response = client.messages.create(
            model=model, max_tokens=1024, messages=messages
        )
        llm_requests_total.labels(model=model, status="success").inc()
        llm_tokens.labels(model=model, type="input").inc(response.usage.input_tokens)
        llm_tokens.labels(model=model, type="output").inc(response.usage.output_tokens)
        return response.content[0].text
    except Exception as e:
        llm_requests_total.labels(model=model, status="error").inc()
        raise
    finally:
        llm_latency.labels(model=model).observe(time.time() - start)
```

---

## Structured Log Analysis

Store logs in a way that's easy to query:

```python
import sqlite3
from contextlib import contextmanager

@contextmanager
def get_db():
    conn = sqlite3.connect("ai_logs.db")
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()

def setup_logs_db():
    with get_db() as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS llm_calls (
                id INTEGER PRIMARY KEY,
                timestamp TEXT,
                session_id TEXT,
                model TEXT,
                input_tokens INTEGER,
                output_tokens INTEGER,
                latency_ms INTEGER,
                user_input TEXT,
                assistant_output TEXT,
                status TEXT,
                error TEXT
            )
        """)
        conn.commit()

def log_call(session_id, model, input_tokens, output_tokens, latency_ms,
             user_input, assistant_output, status, error=None):
    with get_db() as conn:
        conn.execute("""
            INSERT INTO llm_calls VALUES (NULL, datetime('now'), ?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (session_id, model, input_tokens, output_tokens, latency_ms,
              user_input, assistant_output, status, error))
        conn.commit()

# Analyze usage
def get_daily_stats():
    with get_db() as conn:
        return conn.execute("""
            SELECT
                date(timestamp) as date,
                COUNT(*) as calls,
                SUM(input_tokens + output_tokens) as total_tokens,
                AVG(latency_ms) as avg_latency_ms,
                SUM(CASE WHEN status='error' THEN 1 ELSE 0 END) as errors
            FROM llm_calls
            GROUP BY date(timestamp)
            ORDER BY date DESC
            LIMIT 30
        """).fetchall()
```

---

## Tools Overview

| Tool | Purpose | Cost |
|------|---------|------|
| LangSmith | LangChain tracing + evals | Paid (free tier) |
| Helicone | API proxy, logging, analytics | Paid (free tier) |
| Langfuse | Open source LLM observability | Open source / Cloud |
| Weights & Biases | ML experiment tracking | Paid (free tier) |
| OpenTelemetry | Vendor-neutral tracing | Free |
| Prometheus + Grafana | Metrics + dashboards | Free (self-hosted) |
