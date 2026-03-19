# Module 04 — AI Agents & Orchestration

Agents go beyond single-turn Q&A — they reason, plan, use tools, and complete multi-step tasks autonomously.

---

## Topics

| # | Topic | Status |
|---|-------|--------|
| 1 | [Agent Architectures](./01_agent_architectures.md) | ⬜ |
| 2 | [Memory Systems](./02_memory_systems.md) | ⬜ |
| 3 | [Multi-Agent Systems](./03_multi_agent_systems.md) | ⬜ |
| 4 | [Frameworks — LangChain, LlamaIndex, CrewAI](./04_frameworks.md) | ⬜ |

---

## Learning Goals

- Understand how agents loop through reason → act → observe cycles
- Build a basic agent with tool use from scratch
- Implement short-term and long-term memory for agents
- Understand when and how to use multiple agents
- Know the major frameworks and when to use each

---

## What Makes Something an Agent?

A regular LLM call: `input → LLM → output` (one shot)

An agent:
```
input → LLM → action → observe result → LLM → action → observe → ... → final answer
```

The key difference: **the model decides what to do next based on previous results.**

---

## Hands-On Practice

| Type | File | Description |
|------|------|-------------|
| Exercises | [exercises.md](./exercises.md) | 5 focused coding challenges |
| Project | [project.md](./project.md) | One substantial mini-project |

Complete all topic files before attempting the exercises.
Finish the exercises before starting the project.
