# AI Knowledge Base 🧠

> A structured, hands-on learning path for software developers building AI-powered applications and Generative AI solutions.

![Modules](https://img.shields.io/badge/Modules-7-blue)
![Topics](https://img.shields.io/badge/Topics-29-green)
![Hours](https://img.shields.io/badge/Total%20Hours-~100-orange)
![Level](https://img.shields.io/badge/Level-Intermediate-yellow)
![Language](https://img.shields.io/badge/Language-Python-3776AB?logo=python&logoColor=white)

---

## What Is This?

This knowledge base takes you from **understanding how LLMs work** all the way to **building production-grade, secure, local AI systems** — with theory, working code examples, exercises, and real projects at every step.

```
Learn → Practice → Build
  ↓          ↓        ↓
Topics   Exercises  Project
```

---

## Learning Path

| # | Module | Topics | Theory | Practice | Total | Status |
|---|--------|--------|--------|----------|-------|--------|
| 01 | [Foundations](./Module_01_Foundations/README.md) | 3 | 4 hrs | 6 hrs | **10 hrs** | ⬜ |
| 02 | [Building with LLMs](./Module_02_Building_with_LLMs/README.md) | 5 | 5 hrs | 8 hrs | **13 hrs** | ⬜ |
| 03 | [RAG](./Module_03_RAG/README.md) | 4 | 5 hrs | 9 hrs | **14 hrs** | ⬜ |
| 04 | [Agents & Orchestration](./Module_04_Agents_and_Orchestration/README.md) | 4 | 6 hrs | 10 hrs | **16 hrs** | ⬜ |
| 05 | [GenAI Application Patterns](./Module_05_GenAI_Application_Patterns/README.md) | 4 | 5 hrs | 10 hrs | **15 hrs** | ⬜ |
| 06 | [Production & MLOps](./Module_06_Production_and_MLOps/README.md) | 5 | 7 hrs | 10 hrs | **17 hrs** | ⬜ |
| 07 | [Advanced Topics](./Module_07_Advanced_Topics/README.md) | 4 | 8 hrs | 7 hrs | **15 hrs** | ⬜ |
| | **TOTAL** | **29** | **~40 hrs** | **~60 hrs** | **~100 hrs** | |

---

## Module Breakdown

### 01 — Foundations `10 hrs`
> Understand how LLMs work before you start building.

| File | What You'll Learn |
|------|------------------|
| [How LLMs Work](./Module_01_Foundations/01_how_llms_work.md) | Transformers, tokens, embeddings, context windows, temperature |
| [Prompt Engineering](./Module_01_Foundations/02_prompt_engineering.md) | Zero-shot, few-shot, chain-of-thought, system prompts, anti-patterns |
| [AI APIs Overview](./Module_01_Foundations/03_ai_apis_overview.md) | Anthropic, OpenAI, Google, pricing, choosing the right API |
| [Exercises](./Module_01_Foundations/exercises.md) | Token counter, temperature explorer, prompt technique comparison |
| [Project: Prompt Lab CLI](./Module_01_Foundations/project.md) | Terminal tool to test prompts with live cost tracking |

---

### 02 — Building with LLMs `13 hrs`
> Make real API calls, handle errors, stream responses, and control costs.

| File | What You'll Learn |
|------|------------------|
| [Calling LLM APIs](./Module_02_Building_with_LLMs/01_calling_llm_apis.md) | Auth, multi-turn chat, error handling, async calls |
| [Streaming Responses](./Module_02_Building_with_LLMs/02_streaming_responses.md) | SSE, WebSocket streaming, FastAPI integration |
| [Structured Output](./Module_02_Building_with_LLMs/03_structured_output.md) | JSON mode, Pydantic validation, response prefilling |
| [Function Calling & Tool Use](./Module_02_Building_with_LLMs/04_function_calling_tool_use.md) | Tool definitions, parallel calls, the tool loop |
| [Token Management](./Module_02_Building_with_LLMs/05_token_management.md) | Cost optimization, prompt caching, model routing |
| [Exercises](./Module_02_Building_with_LLMs/exercises.md) | Retry logic, streaming stats, async batch processing |
| [Project: Terminal AI Assistant](./Module_02_Building_with_LLMs/project.md) | Streaming assistant with tools and session cost tracking |

---

### 03 — RAG (Retrieval-Augmented Generation) `14 hrs`
> Connect LLMs to your private data — no fine-tuning required.

| File | What You'll Learn |
|------|------------------|
| [Vector Databases](./Module_03_RAG/01_vector_databases.md) | Chroma, Pinecone, pgvector — setup and comparison |
| [Embeddings & Semantic Search](./Module_03_RAG/02_embeddings_semantic_search.md) | Embedding models, cosine similarity, hybrid search |
| [Chunking Strategies](./Module_03_RAG/03_chunking_strategies.md) | Fixed, paragraph, recursive, semantic chunking |
| [RAG Pipeline](./Module_03_RAG/04_rag_pipeline.md) | End-to-end pipeline, reranking, HyDE, LangChain RAG |
| [Exercises](./Module_03_RAG/exercises.md) | Embed & visualize clusters, chunking comparison, retrieval eval |
| [Project: Personal Knowledge Base](./Module_03_RAG/project.md) | CLI Q&A tool over your own PDF/Markdown documents |

---

### 04 — Agents & Orchestration `16 hrs`
> Build AI agents that reason, plan, use tools, and complete multi-step tasks.

| File | What You'll Learn |
|------|------------------|
| [Agent Architectures](./Module_04_Agents_and_Orchestration/01_agent_architectures.md) | ReAct, Plan-and-Execute, Reflection, safety guardrails |
| [Memory Systems](./Module_04_Agents_and_Orchestration/02_memory_systems.md) | In-context, summarization, semantic, episodic memory |
| [Multi-Agent Systems](./Module_04_Agents_and_Orchestration/03_multi_agent_systems.md) | Orchestrator, parallel, pipeline, critic-actor patterns |
| [Frameworks](./Module_04_Agents_and_Orchestration/04_frameworks.md) | LangChain, LlamaIndex, CrewAI — when to use each |
| [Exercises](./Module_04_Agents_and_Orchestration/exercises.md) | Calculator agent, memory agent, 3-stage pipeline |
| [Project: Research Agent](./Module_04_Agents_and_Orchestration/project.md) | Autonomous agent that researches a topic and writes a report |

---

### 05 — GenAI Application Patterns `15 hrs`
> Build the most common AI app types from scratch.

| File | What You'll Learn |
|------|------------------|
| [Chatbots](./Module_05_GenAI_Application_Patterns/01_chatbots.md) | Persona, guardrails, WebSocket chatbot with FastAPI |
| [Document Q&A](./Module_05_GenAI_Application_Patterns/02_document_qa.md) | PDF loading, source citations, REST API design |
| [Code Generation](./Module_05_GenAI_Application_Patterns/03_code_generation.md) | Code gen, review, test gen, SQL generation, refactoring |
| [Multimodal](./Module_05_GenAI_Application_Patterns/04_multimodal.md) | Vision (Claude), Whisper transcription, DALL-E, PDF vision |
| [Exercises](./Module_05_GenAI_Application_Patterns/exercises.md) | Persona bot, receipt scanner, SQL validator, meeting summarizer |
| [Project: Full-Stack AI Web App](./Module_05_GenAI_Application_Patterns/project.md) | Browser app: upload docs → streaming chat with citations |

---

### 06 — Production & MLOps `17 hrs`
> Run AI applications reliably, safely, and efficiently in production.

| File | What You'll Learn |
|------|------------------|
| [Evaluation & Testing](./Module_06_Production_and_MLOps/01_evaluation_testing.md) | Unit evals, LLM-as-judge, RAG eval, regression testing |
| [Observability & Tracing](./Module_06_Production_and_MLOps/02_observability_tracing.md) | Structured logging, LangSmith, OpenTelemetry, Prometheus |
| [Guardrails & Safety](./Module_06_Production_and_MLOps/03_guardrails_safety.md) | Prompt injection defense, DLP, rate limiting, OWASP Top 10 |
| [Caching & Scaling](./Module_06_Production_and_MLOps/04_caching_scaling.md) | Semantic cache, Redis, Anthropic prompt cache, async queues |
| [Fine-Tuning](./Module_06_Production_and_MLOps/05_fine_tuning.md) | When to fine-tune, OpenAI ft API, LoRA/QLoRA on open models |
| [Exercises](./Module_06_Production_and_MLOps/exercises.md) | Eval suite, A/B testing, rate limiter, logging wrapper |
| [Project: Production-Ready AI Service](./Module_06_Production_and_MLOps/project.md) | Harden Module 05 app with guardrails, cache, and monitoring |

---

### 07 — Advanced Topics `15 hrs`
> Cutting-edge tools for experienced AI developers.

| File | What You'll Learn |
|------|------------------|
| [Model Context Protocol (MCP)](./Module_07_Advanced_Topics/01_mcp.md) | Build MCP servers, connect tools to Claude Desktop |
| [Local Models with Ollama](./Module_07_Advanced_Topics/02_local_models.md) | Run Llama, Phi, Mistral locally — fully offline RAG |
| [Open-Source Models](./Module_07_Advanced_Topics/03_open_source_models.md) | Llama 3, Mistral, Phi-4, Qwen — Groq, Together AI, HuggingFace |
| [AI Security](./Module_07_Advanced_Topics/04_ai_security.md) | Prompt injection, jailbreaks, data leakage, OWASP LLM Top 10 |
| [Exercises](./Module_07_Advanced_Topics/exercises.md) | Local model benchmark, MCP server, prompt injection lab |
| [Project: Private Local AI Assistant](./Module_07_Advanced_Topics/project.md) | 100% offline AI assistant — no API keys, no cloud |

---

## Project Progression

Each project builds directly on the previous:

```
01  Prompt Lab CLI
      ↓  add streaming + tools
02  Terminal AI Assistant
      ↓  add document knowledge
03  Personal Knowledge Base
      ↓  add autonomous reasoning
04  Research Agent
      ↓  add web frontend
05  Full-Stack AI Web App
      ↓  add production hardening
06  Production-Ready AI Service
      ↓  go fully offline
07  Private Local AI Assistant ← final capstone
```

---

## How to Use This Repo

```
For each module:
  1. Read all topic .md files     (Theory)
  2. Complete exercises.md        (Focused practice)
  3. Build the project.md         (Hands-on project)
  4. Mark module ✅ in the table
```

---

## Suggested Study Schedule

| Schedule | Hours/Week | Completion |
|----------|-----------|------------|
| Intensive | 20 hrs/week | ~5 weeks |
| Regular | 10 hrs/week | ~10 weeks |
| Part-time | 5 hrs/week | ~20 weeks |

---

## Prerequisites

- Python (functions, classes, async basics)
- Basic REST API knowledge
- Familiarity with JSON
- A terminal and VS Code

---

## Setup

```bash
# Core AI packages
pip install anthropic openai

# RAG
pip install chromadb sentence-transformers tiktoken rank-bm25

# Frameworks
pip install langchain langchain-anthropic langchain-openai langchain-community

# Web / APIs
pip install fastapi uvicorn python-dotenv pydantic

# Documents
pip install pypdf

# Local models
# Download Ollama → https://ollama.ai
ollama pull llama3.2
ollama pull nomic-embed-text
```

---

## License

MIT — free to use, share, and adapt.
