# AI Knowledge Base — Software Development & GenAI Solutions

A structured learning path for developers building AI-powered applications and generative AI solutions.

---

## Overview

| | |
|-|-|
| **Total Modules** | 7 |
| **Total Topics** | 29 |
| **Total Hours (Theory)** | ~40 hrs |
| **Total Hours (Practice)** | ~60 hrs |
| **Total Hours (Full Path)** | **~100 hrs** |
| **Level** | Intermediate Developer |
| **Primary Language** | Python |

---

## Learning Path

| # | Module | Topics | Theory | Practice | Total | Status |
|---|--------|--------|--------|----------|-------|--------|
| 01 | [Foundations](./Module_01_Foundations/README.md) | 3 | 4 hrs | 6 hrs | **10 hrs** | ⬜ Not Started |
| 02 | [Building with LLMs](./Module_02_Building_with_LLMs/README.md) | 5 | 5 hrs | 8 hrs | **13 hrs** | ⬜ Not Started |
| 03 | [RAG](./Module_03_RAG/README.md) | 4 | 5 hrs | 9 hrs | **14 hrs** | ⬜ Not Started |
| 04 | [Agents & Orchestration](./Module_04_Agents_and_Orchestration/README.md) | 4 | 6 hrs | 10 hrs | **16 hrs** | ⬜ Not Started |
| 05 | [GenAI Application Patterns](./Module_05_GenAI_Application_Patterns/README.md) | 4 | 5 hrs | 10 hrs | **15 hrs** | ⬜ Not Started |
| 06 | [Production & MLOps](./Module_06_Production_and_MLOps/README.md) | 5 | 7 hrs | 10 hrs | **17 hrs** | ⬜ Not Started |
| 07 | [Advanced Topics](./Module_07_Advanced_Topics/README.md) | 4 | 8 hrs | 7 hrs | **15 hrs** | ⬜ Not Started |
| | **TOTAL** | **29** | **~40 hrs** | **~60 hrs** | **~100 hrs** | |

> **Theory** = reading topic files
> **Practice** = completing exercises + building the project
> Hours are estimates. Your pace may vary based on experience.

---

## Module Breakdown

### Module 01 — Foundations `~10 hrs`
> Understand how LLMs work before building with them.

| File | Description |
|------|-------------|
| [01 How LLMs Work](./Module_01_Foundations/01_how_llms_work.md) | Transformers, tokens, embeddings, context windows |
| [02 Prompt Engineering](./Module_01_Foundations/02_prompt_engineering.md) | Zero-shot, few-shot, CoT, system prompts |
| [03 AI APIs Overview](./Module_01_Foundations/03_ai_apis_overview.md) | Anthropic, OpenAI, Google, pricing comparison |
| [Exercises](./Module_01_Foundations/exercises.md) | Token counter, temperature explorer, prompt comparison |
| [Project: Prompt Lab CLI](./Module_01_Foundations/project.md) | Terminal tool to test prompts with cost tracking |

---

### Module 02 — Building with LLMs `~13 hrs`
> Integrate LLMs into real applications using APIs and SDKs.

| File | Description |
|------|-------------|
| [01 Calling LLM APIs](./Module_02_Building_with_LLMs/01_calling_llm_apis.md) | Auth, multi-turn chat, error handling, async |
| [02 Streaming Responses](./Module_02_Building_with_LLMs/02_streaming_responses.md) | SSE, WebSockets, streaming in web apps |
| [03 Structured Output](./Module_02_Building_with_LLMs/03_structured_output.md) | JSON mode, Pydantic validation, prefilling |
| [04 Function Calling & Tool Use](./Module_02_Building_with_LLMs/04_function_calling_tool_use.md) | Tool definitions, parallel calls, patterns |
| [05 Token Management](./Module_02_Building_with_LLMs/05_token_management.md) | Cost optimization, caching, model routing |
| [Exercises](./Module_02_Building_with_LLMs/exercises.md) | Retry logic, streaming stats, async batch, JSON extraction |
| [Project: Terminal AI Assistant](./Module_02_Building_with_LLMs/project.md) | Streaming assistant with tool use + cost tracking |

---

### Module 03 — RAG `~14 hrs`
> Connect LLMs to your private data using retrieval-augmented generation.

| File | Description |
|------|-------------|
| [01 Vector Databases](./Module_03_RAG/01_vector_databases.md) | Chroma, Pinecone, pgvector, comparison |
| [02 Embeddings & Semantic Search](./Module_03_RAG/02_embeddings_semantic_search.md) | Embedding models, cosine similarity, hybrid search |
| [03 Chunking Strategies](./Module_03_RAG/03_chunking_strategies.md) | Fixed, paragraph, recursive, semantic chunking |
| [04 RAG Pipeline](./Module_03_RAG/04_rag_pipeline.md) | End-to-end pipeline, LangChain RAG, reranking, HyDE |
| [Exercises](./Module_03_RAG/exercises.md) | Embed & visualize, chunking comparison, hybrid search |
| [Project: Personal Knowledge Base](./Module_03_RAG/project.md) | CLI tool to Q&A over your own documents |

---

### Module 04 — Agents & Orchestration `~16 hrs`
> Build AI agents that reason, plan, and complete multi-step tasks.

| File | Description |
|------|-------------|
| [01 Agent Architectures](./Module_04_Agents_and_Orchestration/01_agent_architectures.md) | ReAct, Plan-and-Execute, Reflection, guardrails |
| [02 Memory Systems](./Module_04_Agents_and_Orchestration/02_memory_systems.md) | In-context, summary, semantic, episodic memory |
| [03 Multi-Agent Systems](./Module_04_Agents_and_Orchestration/03_multi_agent_systems.md) | Orchestrator, parallel, pipeline, critic-actor |
| [04 Frameworks](./Module_04_Agents_and_Orchestration/04_frameworks.md) | LangChain, LlamaIndex, CrewAI, raw API trade-offs |
| [Exercises](./Module_04_Agents_and_Orchestration/exercises.md) | Calculator agent, memory agent, 3-stage pipeline |
| [Project: Research Agent](./Module_04_Agents_and_Orchestration/project.md) | Autonomous agent that researches and writes reports |

---

### Module 05 — GenAI Application Patterns `~15 hrs`
> Build common real-world AI application types end-to-end.

| File | Description |
|------|-------------|
| [01 Chatbots](./Module_05_GenAI_Application_Patterns/01_chatbots.md) | Persona, guardrails, FastAPI + WebSocket |
| [02 Document Q&A](./Module_05_GenAI_Application_Patterns/02_document_qa.md) | PDF loading, source citations, FastAPI API |
| [03 Code Generation](./Module_05_GenAI_Application_Patterns/03_code_generation.md) | Code gen, review, test gen, SQL, refactoring |
| [04 Multimodal](./Module_05_GenAI_Application_Patterns/04_multimodal.md) | Vision (Claude), Whisper audio, DALL-E, PDFs |
| [Exercises](./Module_05_GenAI_Application_Patterns/exercises.md) | Persona bot, receipt scanner, SQL generator, meeting transcriber |
| [Project: Full-Stack AI Web App](./Module_05_GenAI_Application_Patterns/project.md) | Browser-based doc upload + chat with streaming |

---

### Module 06 — Production & MLOps `~17 hrs`
> Run AI apps reliably in production.

| File | Description |
|------|-------------|
| [01 Evaluation & Testing](./Module_06_Production_and_MLOps/01_evaluation_testing.md) | Unit evals, LLM-as-judge, RAG eval, regression |
| [02 Observability & Tracing](./Module_06_Production_and_MLOps/02_observability_tracing.md) | Logging, LangSmith, OpenTelemetry, Prometheus |
| [03 Guardrails & Safety](./Module_06_Production_and_MLOps/03_guardrails_safety.md) | Prompt injection, DLP, rate limiting, OWASP |
| [04 Caching & Scaling](./Module_06_Production_and_MLOps/04_caching_scaling.md) | Semantic cache, Redis, Anthropic prompt cache, async |
| [05 Fine-Tuning](./Module_06_Production_and_MLOps/05_fine_tuning.md) | When to fine-tune, OpenAI ft, LoRA/QLoRA |
| [Exercises](./Module_06_Production_and_MLOps/exercises.md) | Eval suite, A/B testing, rate limiter, logging |
| [Project: Production-Ready AI Service](./Module_06_Production_and_MLOps/project.md) | Add guardrails, caching, monitoring to Module 05 app |

---

### Module 07 — Advanced Topics `~15 hrs`
> Cutting-edge tools and patterns for experienced AI developers.

| File | Description |
|------|-------------|
| [01 Model Context Protocol](./Module_07_Advanced_Topics/01_mcp.md) | MCP servers, resources, tools, Claude Desktop |
| [02 Local Models](./Module_07_Advanced_Topics/02_local_models.md) | Ollama setup, model selection, local RAG |
| [03 Open-Source Models](./Module_07_Advanced_Topics/03_open_source_models.md) | Llama, Mistral, Phi, Qwen, Groq, Together AI |
| [04 AI Security](./Module_07_Advanced_Topics/04_ai_security.md) | Prompt injection, jailbreaks, DLP, OWASP Top 10 |
| [Exercises](./Module_07_Advanced_Topics/exercises.md) | Local model benchmark, MCP server, injection lab |
| [Project: Private Local AI Assistant](./Module_07_Advanced_Topics/project.md) | 100% offline assistant — no API keys, no cloud |

---

## How to Use This Knowledge Base

```
For each module:
  1. Read all topic .md files          (Theory)
  2. Complete exercises.md             (Practice — focused)
  3. Build the project.md              (Practice — substantial)
  4. Mark module ✅ in the table above
```

Track progress by changing `⬜ Not Started` → `🔄 In Progress` → `✅ Done`

---

## Suggested Study Schedule

| Schedule | Hours/Week | Completion Time |
|----------|-----------|-----------------|
| Intensive | 20 hrs/week | ~5 weeks |
| Regular | 10 hrs/week | ~10 weeks |
| Part-time | 5 hrs/week | ~20 weeks |

---

## Prerequisites

- Comfortable with Python (functions, classes, async basics)
- Basic understanding of REST APIs and HTTP
- Familiarity with JSON
- A terminal / VS Code

---

## Tools & Setup

```bash
# Python packages
pip install anthropic openai langchain langchain-anthropic
pip install chromadb sentence-transformers
pip install fastapi uvicorn python-dotenv pydantic
pip install pypdf tiktoken rank-bm25

# Local models
# Download Ollama from: https://ollama.ai
ollama pull llama3.2
ollama pull nomic-embed-text

# IDE
# VS Code + Python extension + GitHub Copilot (optional)
```

---

## Project Progression

Each project builds on the previous:

```
Module 01: Prompt Lab CLI
    ↓ (add streaming + tool use)
Module 02: Terminal AI Assistant
    ↓ (add document knowledge)
Module 03: Personal Knowledge Base
    ↓ (add autonomous reasoning)
Module 04: Research Agent
    ↓ (add web frontend)
Module 05: Full-Stack AI Web App
    ↓ (add production hardening)
Module 06: Production-Ready AI Service
    ↓ (make it fully local)
Module 07: Private Local AI Assistant
```

By Module 07 you will have built a complete, production-grade, private AI assistant from scratch.
