# Code Generation & AI Coding Tools

## Common Use Cases

- Code generation from natural language
- Code explanation and documentation
- Code review and bug detection
- Test generation
- Code refactoring
- SQL generation from natural language

---

## Code Generation

```python
import anthropic

client = anthropic.Anthropic()

def generate_code(description: str, language: str = "python") -> str:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        system=f"""You are an expert {language} developer.
Generate clean, production-ready code with:
- Proper error handling
- Type hints (for Python)
- Brief comments for non-obvious logic
- No unnecessary code or boilerplate

Return ONLY the code, no explanation.""",
        messages=[{"role": "user", "content": description}]
    )
    return response.content[0].text

# Usage
code = generate_code("A function that validates email addresses using regex")
print(code)
```

---

## Code Explanation

```python
def explain_code(code: str, audience: str = "intermediate") -> str:
    audience_map = {
        "beginner": "Explain each line simply. Avoid jargon.",
        "intermediate": "Explain the key concepts and logic flow.",
        "expert": "Focus on design decisions, edge cases, and performance implications."
    }

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=f"You are a patient programming teacher. {audience_map[audience]}",
        messages=[{
            "role": "user",
            "content": f"Explain this code:\n\n```\n{code}\n```"
        }]
    )
    return response.content[0].text
```

---

## Code Review

```python
def review_code(code: str, language: str = "python") -> dict:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        system="""You are a senior software engineer doing code review.
Analyze for: bugs, security issues, performance problems, style issues, missing error handling.
Return JSON with structure:
{
  "issues": [{"severity": "critical|warning|info", "line": N, "description": "...", "fix": "..."}],
  "summary": "...",
  "score": 0-10
}""",
        messages=[{
            "role": "user",
            "content": f"Review this {language} code:\n\n```{language}\n{code}\n```"
        }]
    )
    import json
    return json.loads(response.content[0].text)

# Usage
issues = review_code("""
def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"
    return db.execute(query)
""")

for issue in issues["issues"]:
    print(f"[{issue['severity'].upper()}] Line {issue.get('line', '?')}: {issue['description']}")
```

---

## Test Generation

```python
def generate_tests(code: str, framework: str = "pytest") -> str:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        system=f"""Generate comprehensive {framework} tests for the given code.
Cover: happy path, edge cases, error cases, boundary values.
Include test for each function/method.
Use descriptive test names.""",
        messages=[{
            "role": "user",
            "content": f"Generate tests for:\n\n```python\n{code}\n```"
        }]
    )
    return response.content[0].text

source_code = """
def divide(a: float, b: float) -> float:
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
"""

tests = generate_tests(source_code)
print(tests)
```

---

## SQL Generation (Text-to-SQL)

```python
def text_to_sql(question: str, schema: str) -> str:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        system="""You are a SQL expert. Convert natural language questions to SQL queries.
Rules:
- Use only tables/columns from the provided schema
- Write safe, efficient queries
- Use parameterized queries where user input is involved
- Return ONLY the SQL query, nothing else""",
        messages=[{
            "role": "user",
            "content": f"""Schema:
{schema}

Question: {question}

SQL Query:"""
        }]
    )
    return response.content[0].text

schema = """
Table: users (id, name, email, created_at, plan)
Table: orders (id, user_id, product, amount, status, created_at)
Table: products (id, name, price, category)
"""

sql = text_to_sql("Show me the top 10 users by total order amount this month", schema)
print(sql)
```

---

## Code Refactoring Assistant

```python
def refactor_code(code: str, goals: list[str]) -> dict:
    goals_str = "\n".join(f"- {g}" for g in goals)

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        system="You are an expert code refactoring assistant.",
        messages=[{
            "role": "user",
            "content": f"""Refactor the following code to achieve these goals:
{goals_str}

Original code:
```python
{code}
```

Return JSON: {{"refactored_code": "...", "changes": ["list of changes made"]}}"""
        }]
    )

    import json
    return json.loads(response.content[0].text)

result = refactor_code(
    code=messy_code,
    goals=["Improve readability", "Add type hints", "Split into smaller functions", "Add docstrings"]
)
print(result["refactored_code"])
print("\nChanges made:")
for change in result["changes"]:
    print(f"  - {change}")
```

---

## AI Coding Assistant (Interactive)

```python
import subprocess
import sys

def run_code_safely(code: str) -> tuple[str, str]:
    """Run code in subprocess, return (stdout, stderr)."""
    try:
        result = subprocess.run(
            [sys.executable, "-c", code],
            capture_output=True,
            text=True,
            timeout=10
        )
        return result.stdout, result.stderr
    except subprocess.TimeoutExpired:
        return "", "Timeout: code took too long"

def coding_assistant():
    """Interactive coding assistant that can run and debug code."""
    history = []

    while True:
        user_input = input("\nYou: ").strip()
        if user_input.lower() == "quit":
            break

        history.append({"role": "user", "content": user_input})

        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=2048,
            system="""You are a Python coding assistant.
When asked to write code:
1. Write the code in a ```python block
2. Explain briefly what it does
When debugging:
1. Identify the error
2. Provide the fix""",
            messages=history
        )

        assistant_msg = response.content[0].text
        history.append({"role": "assistant", "content": assistant_msg})
        print(f"\nAssistant: {assistant_msg}")

        # Optionally execute generated code
        import re
        code_match = re.search(r'```python\n(.*?)\n```', assistant_msg, re.DOTALL)
        if code_match:
            run = input("\nRun this code? (y/n): ").strip().lower()
            if run == "y":
                stdout, stderr = run_code_safely(code_match.group(1))
                output = stdout or stderr
                print(f"Output: {output}")

                # Feed result back to assistant
                if stderr:
                    history.append({"role": "user", "content": f"That produced an error: {stderr}"})
```
