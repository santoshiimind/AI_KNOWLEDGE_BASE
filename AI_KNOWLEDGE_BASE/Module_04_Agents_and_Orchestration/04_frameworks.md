# Agent Frameworks

## Overview

| Framework | Best For | Complexity | Maturity |
|-----------|----------|-----------|---------|
| LangChain | General-purpose, RAG + Agents | Medium | High |
| LlamaIndex | Data/document-heavy apps | Medium | High |
| CrewAI | Multi-agent role-based tasks | Low | Medium |
| AutoGen | Conversational multi-agent | Medium | Medium |
| Claude Agent SDK | Production Anthropic agents | Medium | New |

---

## LangChain

The most popular AI framework. Covers chains, RAG, agents, memory, and more.

```bash
pip install langchain langchain-anthropic langchain-openai
```

### LangChain Agent with Tools

```python
from langchain_anthropic import ChatAnthropic
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.tools import tool
from langchain_core.prompts import ChatPromptTemplate

llm = ChatAnthropic(model="claude-sonnet-4-6")

@tool
def get_weather(location: str) -> str:
    """Get the current weather for a location."""
    return f"Weather in {location}: 22°C, sunny"

@tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression."""
    try:
        return str(eval(expression))
    except:
        return "Invalid expression"

tools = [get_weather, calculate]

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

agent = create_tool_calling_agent(llm, tools, prompt)
executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

result = executor.invoke({"input": "What's the weather in Paris and what is 15 * 7?"})
print(result["output"])
```

### LangChain RAG Chain

```python
from langchain_anthropic import ChatAnthropic
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

llm = ChatAnthropic(model="claude-sonnet-4-6")
vectorstore = Chroma(persist_directory="./chroma_db", embedding_function=OpenAIEmbeddings())
retriever = vectorstore.as_retriever()

rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | ChatPromptTemplate.from_template("Answer using context:\n{context}\n\nQuestion: {question}")
    | llm
    | StrOutputParser()
)

answer = rag_chain.invoke("What is our return policy?")
```

---

## LlamaIndex

Focused on connecting LLMs to data. Excellent for document Q&A and knowledge graphs.

```bash
pip install llama-index llama-index-llms-anthropic llama-index-embeddings-openai
```

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader, Settings
from llama_index.llms.anthropic import Anthropic
from llama_index.embeddings.openai import OpenAIEmbedding

# Configure
Settings.llm = Anthropic(model="claude-sonnet-4-6")
Settings.embed_model = OpenAIEmbedding(model="text-embedding-3-small")

# Load and index documents
documents = SimpleDirectoryReader("./docs").load_data()
index = VectorStoreIndex.from_documents(documents)

# Query
query_engine = index.as_query_engine()
response = query_engine.query("What are the main topics covered?")
print(response)
print(response.source_nodes)  # see which docs were used
```

---

## CrewAI

Define agents as a "crew" with roles, goals, and tasks. Great for structured workflows.

```bash
pip install crewai crewai-tools
```

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool

search_tool = SerperDevTool()

# Define specialized agents
researcher = Agent(
    role="Senior Research Analyst",
    goal="Find and summarize accurate information about the given topic",
    backstory="Expert researcher with attention to detail",
    tools=[search_tool],
    verbose=True
)

writer = Agent(
    role="Content Writer",
    goal="Write clear, engaging articles based on research",
    backstory="Experienced tech writer",
    verbose=True
)

editor = Agent(
    role="Senior Editor",
    goal="Polish content to publication quality",
    backstory="Seasoned editor with high standards",
    verbose=True
)

# Define tasks
research_task = Task(
    description="Research the latest developments in {topic}",
    expected_output="A comprehensive summary with key facts and sources",
    agent=researcher
)

write_task = Task(
    description="Write a 500-word article based on the research",
    expected_output="A well-structured article with introduction, body, and conclusion",
    agent=writer,
    context=[research_task]
)

edit_task = Task(
    description="Review and improve the article",
    expected_output="A polished, publication-ready article",
    agent=editor,
    context=[write_task]
)

# Run the crew
crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, write_task, edit_task],
    process=Process.sequential
)

result = crew.kickoff(inputs={"topic": "AI in healthcare"})
print(result)
```

---

## Claude Agent SDK (Anthropic)

Built specifically for Claude-powered agents with built-in support for computer use and complex tasks.

```bash
pip install anthropic-agent-sdk
```

```python
# SDK provides higher-level abstractions for:
# - Computer use (controlling a browser/desktop)
# - Long-running tasks
# - Human-in-the-loop workflows
# See: https://docs.anthropic.com/en/docs/agents
```

---

## When to Use Which Framework

| Scenario | Recommended |
|---------|-------------|
| Quick prototype | LangChain (most tutorials/examples) |
| Document Q&A app | LlamaIndex |
| Structured team workflows | CrewAI |
| Custom from scratch | Raw API (most control) |
| Anthropic-native production | Claude Agent SDK |
| Research/experimentation | AutoGen |

---

## Raw API vs Framework Trade-offs

**Raw API (recommended for production):**
- Full control, no abstractions
- Better debugging
- No framework version conflicts
- More code to write

**Framework (good for prototyping):**
- Faster to get started
- More integrations built-in
- Abstractions can hide what's happening
- Updates can break your code
