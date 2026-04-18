[简体中文](./README.md) | **English**

# Cursor Best Practices

> Cursor is an AI-powered IDE based on VS Code, featuring three core capabilities: code completion, Chat, and Composer (Agent). Its strength lies in deep editor integration — select code to start a conversation, preview changes in real time right in the editor.

---

## Core Concepts

| Concept | Description | Use Case |
|---------|-------------|----------|
| **Tab Completion** | Context-aware code completion | Speed up daily coding |
| **Chat** | Sidebar conversation, select code to ask questions | Understanding code, Q&A |
| **Composer** | Agent mode, cross-file editing | Complex tasks, refactoring |
| **Rules** | `.cursor/rules/*.md` rule files | Control AI behavior |
| **@ References** | `@file` `@folder` `@web` etc. | Pinpoint context precisely |
| **Notepads** | Reusable context snippets | Context management for complex projects |

---

## Getting Started

### .cursorrules — Project Rules

Create `.cursorrules` in your project root or place rule files under `.cursor/rules/`:

```markdown
# Project Rules

## Tech Stack
- React 18 + TypeScript + Tailwind CSS
- State management: Zustand
- Routing: React Router v6
- Testing: Vitest + Testing Library

## Code Style
- Functional components + Hooks only, no class components
- Define Props with interface, not type
- File names in kebab-case, component names in PascalCase
- One component per file

## Do NOT
- Use the any type
- Use useEffect for data fetching — use TanStack Query instead
- Manipulate the DOM directly — use React refs
```

### @ Reference Tips

```
# Reference a file
@src/components/Button.tsx The Props type on this component is wrong, please fix it

# Reference a folder
@src/api/ Error handling across these endpoints is inconsistent, unify them to...

# Reference documentation
@https://tanstack.com/query/latest Refer to the official docs and refactor the useEffect data fetching to useQuery

# Reference terminal
@terminal Check the error output and help me fix it
```

---

## Prompting Tips

### 1. Breaking Down Large Tasks with Composer

```
Use Composer mode. Execute the following steps:
1. Read all page components under src/pages/ to understand the routing structure
2. Create src/layouts/DashboardLayout.tsx as a unified layout
3. Migrate all page components to use the new layout
4. Verify the route configuration is correct
Pause after each step and wait for my confirmation before continuing.
```

### 2. Select Code and Chat Directly

Select code and press `Cmd+L` (macOS) to open Chat:

```
# Select a complex regex
Explain what this regex does. Are there any edge cases it doesn't cover?

# Select a function
What's the time complexity of this function? Is there a more optimal approach?

# Select some CSS
Rewrite this CSS using Tailwind classes
```

### 3. Managing Complex Context with Notepads

For large projects, create a Notepad to save frequently used context:

```
Notepad: "API Spec"
---
All APIs return this format:
{ code: number, data: T, message: string }

Error codes:
- 400: Bad request
- 401: Unauthenticated
- 403: Forbidden
- 500: Server error

Auth: Bearer Token in Authorization header
```

Reference it in conversation: `@notepad:API Spec Implement the user list endpoint following this format`

---

## Advanced Tips

### Splitting Rules into Multiple Files

```
.cursor/rules/
├── global.md          # Global rules (code style, naming, etc.)
├── react.md           # React-specific rules
├── api.md             # API development rules
├── testing.md         # Testing rules
└── security.md        # Security rules
```

Each file can specify trigger conditions:

```markdown
---
globs: ["src/components/**/*.tsx"]
---

# React Component Rules
- All components must have a displayName
- Props with more than 3 fields must use a separate interface
- Must handle loading and error states
```

### Supercharge Rules with superpowers-zh

Writing rules manually is slow. Use superpowers-zh to install methodologies in one command:

```bash
cd /your/project
npx superpowers-zh
# Auto-installs to .cursor/rules/ with brainstorming, debugging, verification skills, etc.
```

Once installed, Cursor automatically loads the matching skill rules for relevant files.

### Model Selection Strategy

| Scenario | Recommended Model | Why |
|----------|-------------------|-----|
| Tab completion | cursor-small | Fast, low latency |
| Simple Q&A | Claude Sonnet | Best value |
| Complex refactoring | Claude Opus | Strongest comprehension |
| Large Composer tasks | Claude Opus | Best at multi-file coordination |

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Accept completion |
| `Cmd+L` | Open Chat (selected code auto-included) |
| `Cmd+I` | Open Composer |
| `Cmd+K` | Inline edit (after selecting code) |
| `Cmd+Shift+L` | Add current file to Chat context |

---

## Common Pitfalls

| Pitfall | Description | Solution |
|---------|-------------|----------|
| Rules too long | `.cursorrules` is thousands of lines, AI can't retain it all | Split into `.cursor/rules/` with globs for on-demand loading |
| Composer goes off the rails | Agent mode edits files it shouldn't | Use `@file` to limit scope, or mark no-go zones in rules |
| Completion too aggressive | Tab completion generates too much code at once | Adjust completion length in settings, or press `Esc` to reject |
| Not enough context | AI doesn't understand project structure | Use `@folder` to reference key directories, write good `.cursorrules` |

👉 **Deep dive**: [Cursor Pitfalls](../pitfalls/cursor.en.md) — 8 real-world traps (Composer rogue / @file silent fail / stale Notepads and more), each with Symptom / Cause / Recovery / Prevention

---

## Configuration Templates

Copy directly into your project's `.cursor/rules/` directory:

| Template | Purpose |
|----------|---------|
| [global.cursorrules.md](templates/global.cursorrules.md) | Global rules (code style, naming, restrictions) |
| [api.cursorrules.md](templates/api.cursorrules.md) | API development rules (only active in API directories) |

### Model Configuration

Cursor's model selection is configured in the settings UI (`Cursor Settings > Models`) — no need to manually edit JSON.

---

## Further Reading

- [Cursor Official Docs](https://docs.cursor.com)
- [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) — Community rules collection (38k+ stars)
- [superpowers-zh](https://github.com/jnMetaCode/superpowers-zh) — Skills methodology (also supports Cursor)
