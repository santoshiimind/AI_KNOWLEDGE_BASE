# Model Context Protocol (MCP)

## What Is MCP?

MCP (Model Context Protocol) is an open standard by Anthropic that lets AI models connect to external data sources and tools in a standardized way.

Think of it as **USB for AI** — any MCP-compatible tool can plug into any MCP-compatible AI.

- **Without MCP**: each app builds custom integrations
- **With MCP**: build once, use everywhere

---

## MCP Architecture

```
Claude (MCP Host)
      ↕  (MCP Protocol)
MCP Server
      ↕
External Resource (files, databases, APIs, etc.)
```

An MCP server exposes:
- **Resources**: data sources (files, DB rows, API responses)
- **Tools**: functions the model can call
- **Prompts**: reusable prompt templates

---

## Building an MCP Server (Python)

```bash
pip install mcp
```

```python
import mcp.server.stdio
from mcp.server import Server
from mcp.types import Tool, TextContent
import json
import sqlite3

server = Server("my-database-server")

# Define tools
@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="query_database",
            description="Execute a read-only SQL query on the product database",
            inputSchema={
                "type": "object",
                "properties": {
                    "sql": {
                        "type": "string",
                        "description": "SQL SELECT query to execute"
                    }
                },
                "required": ["sql"]
            }
        ),
        Tool(
            name="get_table_schema",
            description="Get the schema of all tables in the database",
            inputSchema={
                "type": "object",
                "properties": {}
            }
        )
    ]

# Implement tools
@server.call_tool()
async def call_tool(name: str, arguments: dict):
    conn = sqlite3.connect("products.db")
    conn.row_factory = sqlite3.Row

    if name == "query_database":
        sql = arguments["sql"]

        # Safety: only allow SELECT
        if not sql.strip().upper().startswith("SELECT"):
            return [TextContent(type="text", text="Error: Only SELECT queries are allowed")]

        cursor = conn.execute(sql)
        rows = [dict(row) for row in cursor.fetchall()]
        return [TextContent(type="text", text=json.dumps(rows, indent=2))]

    elif name == "get_table_schema":
        cursor = conn.execute("SELECT name FROM sqlite_master WHERE type='table'")
        tables = [row[0] for row in cursor.fetchall()]

        schema = {}
        for table in tables:
            cursor = conn.execute(f"PRAGMA table_info({table})")
            schema[table] = [{"name": col[1], "type": col[2]} for col in cursor.fetchall()]

        return [TextContent(type="text", text=json.dumps(schema, indent=2))]

    return [TextContent(type="text", text=f"Unknown tool: {name}")]

# Run the server
if __name__ == "__main__":
    import asyncio
    asyncio.run(mcp.server.stdio.run_server(server))
```

---

## MCP Server with Resources

```python
from mcp.types import Resource

@server.list_resources()
async def list_resources():
    import os
    docs_dir = "./docs"
    resources = []

    for filename in os.listdir(docs_dir):
        if filename.endswith((".txt", ".md")):
            resources.append(Resource(
                uri=f"file://docs/{filename}",
                name=filename,
                description=f"Document: {filename}",
                mimeType="text/plain"
            ))

    return resources

@server.read_resource()
async def read_resource(uri: str):
    filename = uri.replace("file://docs/", "")
    with open(f"./docs/{filename}", "r") as f:
        content = f.read()
    return content
```

---

## Connecting MCP to Claude Desktop

Add to `~/.config/claude/claude_desktop_config.json` (Mac/Linux):
or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "my-database": {
      "command": "python",
      "args": ["/path/to/your/server.py"],
      "env": {
        "DATABASE_PATH": "/path/to/products.db"
      }
    }
  }
}
```

Claude Desktop will now automatically start your MCP server and Claude can use your tools in conversation.

---

## Popular Pre-Built MCP Servers

| Server | Capability |
|--------|-----------|
| `@modelcontextprotocol/server-filesystem` | Read/write local files |
| `@modelcontextprotocol/server-github` | GitHub repos, issues, PRs |
| `@modelcontextprotocol/server-postgres` | PostgreSQL queries |
| `@modelcontextprotocol/server-slack` | Slack messages |
| `@modelcontextprotocol/server-brave-search` | Web search |
| `mcp-server-sqlite` | SQLite databases |

Install and configure in `claude_desktop_config.json`.

---

## When to Build an MCP Server

Build an MCP server when:
- You want Claude to access your private data
- You want to reuse the integration across multiple Claude apps
- You need Claude Desktop to have custom tools
- You're building tools for others to use with Claude

Use tool calling directly when:
- You're building a specific app (not for general reuse)
- You have full control over the execution environment
