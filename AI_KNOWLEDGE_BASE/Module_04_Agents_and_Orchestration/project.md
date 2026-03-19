# Module 04 — Project: Research Agent

## What You'll Build

An autonomous research agent that takes a topic, researches it using multiple tools, and produces a well-structured research report — all without human intervention.

---

## Features to Implement

### Core (Required)
- [ ] Agent loop with ReAct pattern (reason → act → observe)
- [ ] Tools: `search(query)`, `summarize(text)`, `save_note(title, content)`
- [ ] Produces a final structured Markdown report
- [ ] Short-term memory (notes saved during research are available to the agent)
- [ ] Max iterations guard (stop at 15 tool calls)
- [ ] Final report saved to a `.md` file

### Stretch Goals
- [ ] Real web search via SerpAPI or Brave Search API
- [ ] Wikipedia tool: `get_wikipedia_article(title)`
- [ ] Multi-agent: separate Researcher + Writer + Fact-Checker agents
- [ ] Cost tracking: report total cost of the research session
- [ ] Quality self-check: after writing, agent reviews its own report and identifies gaps

---

## Project Structure

```
research_agent/
├── main.py          # entry point: takes topic as argument
├── agent.py         # core ReAct loop
├── tools.py         # tool definitions + implementations
├── notes.py         # in-memory note store (agent's scratchpad)
├── reporter.py      # formats final report from notes
└── reports/         # output directory for generated reports
```

---

## Tools to Implement

```python
TOOLS = [
    {
        "name": "search",
        "description": "Search for information on a topic. Returns a brief summary of findings.",
        # input: query (string)
    },
    {
        "name": "save_note",
        "description": "Save an important finding or fact to your research notes for later use.",
        # input: title (string), content (string)
    },
    {
        "name": "list_notes",
        "description": "List all research notes saved so far.",
        # input: none
    },
    {
        "name": "finish_research",
        "description": "Call this when research is complete. Triggers final report generation.",
        # input: none
    }
]
```

---

## Sample Run

```bash
$ python main.py "Impact of large language models on software development"

[Research Agent Starting]
Topic: Impact of large language models on software development

Iteration 1:
  Thought: I should search for recent developments in LLMs for software dev
  Action: search("LLM impact on software development 2024")
  Observation: Found 3 relevant results...

Iteration 2:
  Thought: I should look into specific tools like GitHub Copilot
  Action: search("GitHub Copilot developer productivity statistics")
  ...

Iteration 8:
  Action: finish_research()

[Generating Report...]
Report saved to: reports/llm_software_development_2024-01-15.md
Total tool calls: 8 | Estimated cost: $0.024
```

---

## Report Template

```markdown
# Research Report: {topic}
Generated: {date}

## Executive Summary
{2-3 sentence overview}

## Key Findings
1. {finding with source}
2. ...

## Detailed Analysis
### {Section 1}
...

## Conclusion
...

## Research Notes
{all saved notes}
```

---

## What You'll Learn

- Building a complete autonomous agent loop
- Tool design and implementation
- Agent memory (notes as working memory)
- Controlling agent behavior with guardrails
- Producing structured output from an agent
