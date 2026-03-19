# 🧠 AI Learning Hub

> A hands-on, structured learning repository for software developers building AI-powered applications and Generative AI solutions.

![Modules](https://img.shields.io/badge/Modules-7-blue)
![Topics](https://img.shields.io/badge/Topics-29-green)
![Hours](https://img.shields.io/badge/Total%20Hours-~100-orange)
![Level](https://img.shields.io/badge/Level-Intermediate-yellow)
![Language](https://img.shields.io/badge/Language-Python-3776AB?logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## 📚 Contents

| Folder | Description |
|--------|-------------|
| [📖 AI_KNOWLEDGE_BASE](./AI_KNOWLEDGE_BASE/README.md) | Structured 7-module learning path — theory, code examples, exercises & projects |

---

## 🗺️ Learning Path — AI_KNOWLEDGE_BASE

A complete curriculum taking you from **LLM fundamentals** to **production-grade AI systems** in ~100 hours.

```
Module 01 — Foundations              (~10 hrs)
  └─ How LLMs work · Prompt Engineering · AI APIs

Module 02 — Building with LLMs       (~13 hrs)
  └─ API calls · Streaming · Tool Use · Token Management

Module 03 — RAG                       (~14 hrs)
  └─ Vector DBs · Embeddings · Chunking · Full Pipeline

Module 04 — Agents & Orchestration    (~16 hrs)
  └─ ReAct · Memory · Multi-Agent · LangChain / CrewAI

Module 05 — GenAI Application Patterns (~15 hrs)
  └─ Chatbots · Document Q&A · Code Gen · Multimodal

Module 06 — Production & MLOps        (~17 hrs)
  └─ Evals · Observability · Guardrails · Caching · Fine-tuning

Module 07 — Advanced Topics            (~15 hrs)
  └─ MCP · Local Models · Open-Source Models · AI Security
```

**Total: 7 modules · 29 topics · ~100 hours**

---

## 🏗️ Project Progression

Every module ends with a hands-on project. Each one builds on the last:

```
[01] Prompt Lab CLI
       ↓
[02] Terminal AI Assistant  (+ streaming + tools)
       ↓
[03] Personal Knowledge Base  (+ RAG over your docs)
       ↓
[04] Research Agent  (+ autonomous reasoning)
       ↓
[05] Full-Stack AI Web App  (+ browser UI)
       ↓
[06] Production-Ready AI Service  (+ guardrails + monitoring)
       ↓
[07] Private Local AI Assistant  (+ 100% offline, no API keys)
```

---

## ⚡ Quick Start

```bash
# Clone the repo
git clone https://github.com/santoshiimind/AI_KNOWLEDGE_BASE.git
cd AI_KNOWLEDGE_BASE

# Install dependencies
pip install anthropic openai chromadb langchain fastapi uvicorn
pip install sentence-transformers pypdf python-dotenv pydantic

# Set your API key
cp .env.example .env   # then add your ANTHROPIC_API_KEY

# Start with Module 01
open AI_KNOWLEDGE_BASE/Module_01_Foundations/README.md
```

---

## 🧩 What's Inside Each Module

Every module contains:

```
Module_XX/
├── README.md          ← overview, goals, time estimate
├── 01_topic.md        ← concept + key terms + code examples
├── 02_topic.md
├── ...
├── exercises.md       ← 5 focused coding challenges
└── project.md         ← one substantial hands-on project
```

---

## 🛠️ Tech Stack Covered

| Category | Tools |
|----------|-------|
| **LLM APIs** | Anthropic Claude, OpenAI GPT, Google Gemini |
| **Frameworks** | LangChain, LlamaIndex, CrewAI |
| **Vector DBs** | ChromaDB, Pinecone, pgvector |
| **Local Models** | Ollama, Llama 3, Phi-4, Mistral, Qwen |
| **Web** | FastAPI, WebSockets, Server-Sent Events |
| **MLOps** | LangSmith, Prometheus, OpenTelemetry |
| **Protocol** | Model Context Protocol (MCP) |

---

## 📋 Prerequisites

- Python basics (functions, classes, async)
- Familiarity with REST APIs and JSON
- A terminal and VS Code

---

## 🤝 Contributing

Found a mistake or want to add content? PRs are welcome!

1. Fork the repo
2. Create a branch: `git checkout -b improve/module-01`
3. Commit your changes
4. Open a Pull Request

---

## 📄 License

MIT — free to use, share, and adapt with attribution.
