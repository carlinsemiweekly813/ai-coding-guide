[简体中文](./cheatsheet.md) | **English**

# 9-Tool Cheatsheet

> Key parameters, commands, and hotkeys for all 9 tools on one page. Models and pricing move fast — check each tool's website for the current truth.
>
> Snapshot date: **2026-04**

---

## 1. Pick a Tool in 30 Seconds

| What you want | First pick | Alternatives |
|---------------|------------|--------------|
| Tab completion inside an IDE | **Cursor** | Copilot / Windsurf / Trae |
| Agent in the terminal for complex tasks | **Claude Code** | Aider / Gemini CLI |
| Analyze a huge codebase in one pass | **Gemini CLI** (2M context) | Aider + map mode |
| Tight budget / zero cost | **Trae** (free) or **Gemini CLI** (free tier) | Aider + local models |
| Direct network in China (no VPN) | **Trae** | OpenClaw + local model |
| Team collaboration, spec-driven | **Kiro** (Spec) | Claude Code plan mode |
| Git-native, multi-model | **Aider** | — |
| AI agent automation (beyond coding) | **OpenClaw** | — |
| Stick with VS Code, nothing new to install | **Copilot** | Cursor/Windsurf/Trae are VS Code forks |

---

## 2. Capability Matrix

| Dimension | Claude Code | Cursor | Copilot | Windsurf | Gemini CLI | Kiro | Aider | Trae | OpenClaw |
|-----------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Type** | CLI | IDE | IDE plugin | IDE | CLI | IDE | CLI | IDE | Agent framework |
| **Vendor** | Anthropic | Cursor | GitHub | Codeium | Google | AWS | OSS | ByteDance | OSS |
| **Tab completion** | — | ★★★ | ★★★ | ★★★ | — | ★★ | — | ★★ | — |
| **Agent execution** | ★★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★ | ★★ | ★★ | ★★★ |
| **Runs in terminal** | ★★★ | ★ | — | ★ | ★★★ | — | ★★★ | — | ★★★ |
| **Context window** | 200K | per model | per model | per model | **2M** | per model | per model | per model | per model |
| **MCP support** | ✅ | ✅ | ✅ | ✅ | Extension-based | — | — | — | Native |
| **Hook automation** | ✅ | — | — | — | — | ✅ | ✅ (lint/test) | — | ✅ (Cron) |
| **Subagent** | ✅ | — | — | — | — | — | — | — | ✅ (workspaces) |
| **Multi-model** | ★ (Claude only) | ★★★ | ★★ | ★★★ | ★ (Gemini only) | ★★ | ★★★ (almost any LLM) | ★★ | ★★★ |
| **Open source** | — | — | — | — | — | — | ✅ MIT | — | ✅ MIT |
| **Direct access in China** | — | — | — | — | — | — | — | ✅ | ✅ (with local model) |
| **Pricing model** | Per-API / Pro plan | Free / $20 Pro | Free / $10 Pro | Free / paid plan | Generous free tier | Free preview | Whatever LLM you pick | Free (with quota) | OSS free |

---

## 3. Config Files at a Glance

Knowing where each tool keeps its config is half the onboarding battle.

| Tool | Main config | Location | Notes |
|------|------------|----------|-------|
| Claude Code | `CLAUDE.md` + `.claude/` | Project root | Keep under 200 lines; split to `.claude/rules/` for big projects |
| Cursor | `.cursor/rules/*.md` | Project root | Supports `globs` for file-type loading |
| Copilot | `.github/copilot-instructions.md` | Project root | Plus `agents/` and `chatModes/` in same dir |
| Windsurf | `.windsurfrules` | Project root | Single file, no splitting |
| Gemini CLI | `GEMINI.md` | Project root | Structure similar to CLAUDE.md |
| Kiro | `.kiro/steering/*.md` | Project root | Three modes: `always` / `globs` / `manual` |
| Aider | `.aider.conf.yml` | Project root | YAML with model/lint/test settings |
| Trae | `.trae/rules/project_rules.md` | Project root | Chinese rules supported |
| OpenClaw | `~/.openclaw/openclaw.json` | User home | JSON5, hot-reloaded |

---

## 4. Essential Commands

### Claude Code

```bash
# Install & run
npm install -g @anthropic-ai/claude-code
claude                          # Enter interactive
claude --resume                 # Resume last session
claude --model haiku            # Cheap model for simple tasks
claude -p "task" --output-format json   # Headless

# Inside the session
/compact                        # Compress context
/plan                           # Enter plan mode
Esc Esc                         # Roll back to checkpoint
```

### Cursor

```
Tab             — Accept completion
Cmd+L           — Open Chat (selection auto-attached)
Cmd+I           — Open Composer (agent mode)
Cmd+K           — Inline edit
Cmd+Shift+L     — Add current file to Chat context
Esc             — Reject completion

@file @folder @web @terminal     — References
@notepad:name                    — Reference a Notepad
```

### GitHub Copilot

```
Tab             — Accept completion
Esc             — Reject completion
Cmd+Shift+I     — Open Chat
Cmd+I           — Inline edit
Alt+] / Alt+[   — Cycle completion suggestions

#file #selection #terminal #problems   — References
@workspace                              — Whole-project context
```

### Windsurf

```
Write mode      — Direct code changes (like Composer)
Chat mode       — Q&A

@file @folder @web     — References
Cascade tracks your edit flow automatically — no need to paste context
```

### Gemini CLI

```bash
npm install -g @google/gemini-cli
gemini                          # Enter interactive
# Use the 2M context for big-codebase analysis (overkill for small tasks)
```

### Kiro

```
1. Describe need → 2. Kiro writes Spec → 3. You review → 4. Auto-implement + test

Steering load modes:
- always            Loaded every conversation
- globs: ["*.java"] Loaded for matching files
- manual            Invoked on demand
```

### Aider

```bash
# Install & run
pip install aider-chat
aider --model claude-3-5-sonnet
aider --model deepseek/deepseek-chat   # Cheap
aider --model ollama/qwen2.5-coder     # Free, local

# Inside the session
/add file1 file2      — Add files to context
/drop file            — Remove file
/code                 — Code mode (edit directly)
/ask                  — Ask mode (no edits)
/architect            — Design first, then implement
```

### Trae

```
Builder mode    — Agent, multi-file edits
Chat mode       — Q&A

@file @folder @web     — References
Free models: Claude / GPT, pick per task complexity
```

### OpenClaw

```bash
openclaw onboard                    # Initial setup
openclaw gateway start              # Start the gateway
openclaw doctor                     # Diagnostics

openclaw skills install <slug>      # Install a Skill
openclaw cron add "0 9 * * *" "..."  # Scheduled task
openclaw models set default <model>  # Switch model
openclaw channels add telegram       # Add messaging channel
```

---

## 5. Decision Flow

```
┌─ Mostly working in a terminal?
│   ├─ Need strongest Agent    → Claude Code
│   ├─ Need huge context / free → Gemini CLI
│   └─ Multi-model / Git-native → Aider
│
├─ Mostly in an IDE?
│   ├─ Don't want a new IDE    → GitHub Copilot (VS Code/JetBrains plugin)
│   ├─ Willing to switch, budget ok → Cursor
│   ├─ Want the AI to be proactive  → Windsurf
│   ├─ Chinese UI / local network / free → Trae
│   └─ Team / spec-driven      → Kiro
│
└─ Non-coding AI automation?
    └─ Multi-platform, cron, skills → OpenClaw
```

---

## 6. Recommended Combos

| Combo | When |
|-------|------|
| **Claude Code + Cursor** | Most popular full-stack combo: CLI for heavy lifting, IDE for daily work |
| **Claude Code + Copilot** | Lightweight for pure VS Code users |
| **Gemini CLI + Cursor** | Budget-conscious: 2M free analysis + $20 IDE |
| **Aider + local LLM** | Zero API cost: `ollama/qwen2.5-coder` + Aider |
| **Claude Code + OpenClaw** | Coding + automation: CC codes, OpenClaw runs scheduled tasks |

See [Tool Selection Guide](workflows/tool-selection.en.md) and [Real-World Scenarios](workflows/scenarios.en.md) for details.

---

## 7. Further Reading

- Per-tool deep dives: each tool's README at the project root
- Prompt techniques: [common/prompting.en.md](common/prompting.en.md)
- Methodology overview: see the [Universal Methodologies](README.en.md#universal-methodologies) section in the main README
