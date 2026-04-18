[简体中文](./README.md) | **English**

# GitHub Copilot Best Practices

> GitHub Copilot is the built-in AI coding assistant for VS Code and JetBrains IDEs. Its core strength is **seamless integration** — no tool switching needed, just write code in your editor and get completions and suggestions naturally. Copilot Agent mode evolves it from a completion tool into an autonomous agent that can tackle tasks independently.

---

## Core Concepts

| Concept | Description | Use Case |
|---------|-------------|----------|
| **Code Completion** | Inline gray suggestions | Daily coding, press Tab to accept |
| **Chat** | Sidebar conversation `Cmd+Shift+I` | Q&A, code explanations |
| **Agent Mode** | Autonomously completes multi-step tasks | Complex tasks, cross-file changes |
| **Copilot Instructions** | `.github/copilot-instructions.md` | Project-level configuration |
| **Chat Modes** | Custom conversation roles | Security auditor, testing expert, etc. |
| **MCP Server** | Extends Copilot's tool capabilities | Connect to databases, APIs, etc. |
| **# References** | `#file` `#selection` `#terminal` | Pinpoint context precisely |

---

## Getting Started

### Copilot Instructions — Project Configuration

Create `.github/copilot-instructions.md` in your project root:

```markdown
# Project Guidelines

## Tech Stack
- Python 3.12 + FastAPI + SQLAlchemy 2.0
- Database: PostgreSQL
- Cache: Redis
- Testing: pytest

## Code Standards
- Type annotations must be complete, using Python 3.10+ syntax
- Prefix all async functions with `async_`
- All endpoints must use Pydantic models for input validation
- Use custom exception classes for errors, never bare Exception

## Directory Conventions
- src/api/     — FastAPI routes
- src/models/  — SQLAlchemy models
- src/schemas/ — Pydantic models
- src/services/ — Business logic
- tests/       — Tests (mirrors src structure)
```

### # Reference Tips

```
# Reference a file
#file:src/models/user.py Write a user registration endpoint based on this model

# Reference selected code
#selection What performance issues does this code have?

# Reference terminal output
#terminal Check the error and help me fix it

# Reference VS Code problems panel
#problems Fix these type errors for me
```

---

## Agent Mode

Copilot Agent was the biggest update of 2025. Enter a task in the chat, and Agent will:

1. Analyze requirements -> 2. Search relevant files -> 3. Create a plan -> 4. Execute step by step -> 5. Run tests to verify

### Using Agent Mode

Select Agent mode in Chat (or just describe a complex task):

```
@workspace Add rate limiting to all endpoints under src/api/,
using Redis as the counter. Max 60 requests per user per minute.
Requirements:
1. A reusable rate_limit decorator
2. Redis connection config
3. Return 429 status when limit exceeded
4. Add tests for each endpoint
```

### Agent + MCP for Extended Capabilities

Use MCP Servers to give Agent access to external tools:

```json
// .vscode/settings.json
{
  "github.copilot.chat.mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-postgres", "postgresql://..."]
    }
  }
}
```

Then you can say:

```
Look up the structure of the users table in the database,
then generate a SQLAlchemy model and Pydantic schema based on the actual columns.
```

---

## Prompting Tips

### 1. Write Good Comments for Better Completions

```python
# Completion quality depends on context. A single comment tells Copilot what you need:

# Count the number of business days between two dates, excluding weekends and public holidays
def count_business_days(start: date, end: date) -> int:
    # Copilot will auto-complete the full implementation
```

### 2. Custom Agents for Specialized Reviews

Create `.github/agents/security-reviewer.agent.md`:

```markdown
---
name: Security Reviewer
description: Review code security against OWASP Top 10
---

You are a security review expert. When reviewing code:
1. Check against each item in the OWASP Top 10
2. Focus on SQL injection, XSS, CSRF, and authorization bypass
3. Check for sensitive data stored in plaintext or leaked in logs
4. Label risk levels: 🔴 Critical / 🟡 Medium / 🟢 Low
```

Invoke this role in Chat with `@security-reviewer`.

### 3. Leverage Workspace for Global Understanding

```
@workspace Are there any hardcoded secrets or sensitive values in the project?
Find them all and refactor to use environment variables.
```

---

## Advanced Tips

### Custom Instruction Files

Copilot supports instruction files to fine-tune behavior:

- **`.github/copilot-instructions.md`** — Global project instructions
- **`.github/chatModes/`** — Custom Chat roles (e.g., security reviewer, testing expert)

If you use superpowers-zh, you can also write methodology in skill files under `.claude/skills/` — Copilot Agent's `@workspace` command reads markdown files in the project.

### VS Code Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Accept completion |
| `Esc` | Reject completion |
| `Cmd+Shift+I` | Open Copilot Chat |
| `Cmd+I` | Inline edit |
| `Alt+]` / `Alt+[` | Cycle through completion suggestions |

### Completion Optimization

Tips for more accurate Copilot completions:

1. **Keep related files open** — Copilot reads open tabs as context
2. **Write thorough type annotations** — More complete types = more accurate completions
3. **Write function signatures first** — Define the name, parameters, and return type, then let Copilot complete the body
4. **Keep files short** — Large files add context noise, causing Copilot to drift

---

## Common Pitfalls

| Pitfall | Description | Solution |
|---------|-------------|----------|
| Outdated API completions | Copilot suggests deprecated library APIs | Open the library source or docs as a tab for context |
| Agent edits wrong files | Multi-file changes touch things they shouldn't | Use `#file` to limit scope |
| Ignores .gitignore | Copilot may read node_modules | Check exclude settings in configuration |
| Instructions too long | Exceeds model context window | Trim to essential rules, move details into Chat Modes |

👉 **Deep dive**: [Copilot Pitfalls](../pitfalls/copilot.en.md) — 8 real-world traps (Agent reads .env / MCP silent fail / free tier throttling and more), each with Symptom / Cause / Recovery / Prevention

---

## Configuration Templates

Copy directly into your project:

| Template | Purpose |
|----------|---------|
| [copilot-instructions.md](templates/copilot-instructions.md) | Project guidelines template, copy to `.github/copilot-instructions.md` |
| [security-reviewer.agent.md](templates/security-reviewer.agent.md) | Security reviewer role, copy to `.github/agents/` |

---

## Further Reading

- [Copilot Official Docs](https://docs.github.com/en/copilot)
- [awesome-copilot](https://github.com/github/awesome-copilot) — Official resource collection (27k+ stars)
- [superpowers-zh](https://github.com/jnMetaCode/superpowers-zh) — Skills methodology (also supports VS Code Copilot)
