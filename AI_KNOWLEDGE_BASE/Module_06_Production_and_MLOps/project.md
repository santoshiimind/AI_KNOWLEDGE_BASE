# Module 06 — Project: Production-Ready AI Service

## What You'll Build

Take the web app from Module 05 and transform it into a production-ready service with observability, safety guardrails, caching, and automated quality evaluation.

---

## Features to Implement

### Core (Required)
- [ ] **Structured logging**: every request/response logged to SQLite with full metadata
- [ ] **Rate limiting**: 20 requests/minute per session, with clear error messages
- [ ] **Input guardrails**: detect and block prompt injection + harmful requests
- [ ] **Output guardrails**: scan responses for sensitive data before sending
- [ ] **Semantic cache**: cache similar questions with 0.93 similarity threshold
- [ ] **Admin dashboard**: `/admin` page showing usage stats, top questions, error rate

### Stretch Goals
- [ ] Automated nightly eval run (pick 10 random logged questions, score them, email report)
- [ ] Cost budget: if daily cost exceeds $1.00, switch to a cheaper model automatically
- [ ] Prometheus metrics endpoint (`/metrics`) for Grafana integration
- [ ] Retry queue: failed LLM calls go to a background retry queue
- [ ] Health check endpoint with dependency checks (vector DB, LLM API)

---

## Project Structure

```
ai_service/
├── main.py               # FastAPI app
├── middleware/
│   ├── rate_limiter.py   # per-session rate limiting
│   ├── guardrails.py     # input + output safety
│   └── logger.py         # structured request logging
├── cache/
│   └── semantic_cache.py # embedding-based response cache
├── eval/
│   └── auto_eval.py      # automated quality scoring
├── rag/
│   ├── indexer.py
│   └── retriever.py
├── dashboard/
│   └── index.html        # admin stats page
└── database/
    └── logs.db           # SQLite log store
```

---

## Admin Dashboard Design

```
/admin

=== AI Service Dashboard ===
Period: Last 24 hours

USAGE
  Total requests:     247
  Unique sessions:    38
  Cache hit rate:     23%
  Avg latency:        1,847ms

QUALITY
  Avg LLM judge score: 4.1/5
  Failed requests:     3 (1.2%)
  Blocked requests:    7 (2.8%)
    - Rate limited:    4
    - Guardrail:       3

COST
  Total tokens:       482,130
  Estimated cost:     $0.87
  Budget remaining:   $0.13

TOP QUESTIONS (last 24h)
  1. "What is the refund policy?" (12 times)
  2. "How do I contact support?" (8 times)
  ...
```

---

## Middleware Stack Order

```python
# main.py — wrap every request through this pipeline
@app.middleware("http")
async def pipeline(request: Request, call_next):
    # 1. Rate limiter check
    # 2. Input guardrail check
    # 3. Check semantic cache
    # 4. Process request
    # 5. Output guardrail check
    # 6. Log to DB
    # 7. Return response
    pass
```

---

## Evaluation Automation

```python
# eval/auto_eval.py
# Run this as a cron job or scheduled task

def run_daily_eval():
    # 1. Fetch 10 random Q&A pairs from logs
    pairs = get_random_logged_pairs(n=10)

    scores = []
    for pair in pairs:
        score = llm_judge(pair["question"], pair["answer"])
        scores.append(score)

    report = {
        "date": today(),
        "avg_score": mean(scores),
        "min_score": min(scores),
        "low_quality_examples": [p for p, s in zip(pairs, scores) if s < 3]
    }

    save_eval_report(report)
    # Optionally: send email/Slack alert if avg_score drops
    if report["avg_score"] < 3.5:
        alert(f"Quality alert: avg score dropped to {report['avg_score']}")
```

---

## What You'll Learn

- Building production middleware (rate limiting, guardrails, logging)
- Semantic caching as a cost optimization
- Automated quality monitoring
- Operational dashboards for AI services
- The difference between a demo and a production app
