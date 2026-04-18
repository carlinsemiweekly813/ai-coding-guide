[简体中文](./copilot.md) | **English**

# GitHub Copilot Pitfalls

> Copilot is the oldest AI coding tool and most underestimated — people think "it's just completion," but Agent mode and MCP support are deep. 8 real-world pitfalls.
>
> Hit a new one? Open an Issue or PR.

---

## Pitfall 1: Completion Uses Outdated APIs

**Symptom**
- Generated code uses an old library API
- You paste it and get `Deprecated: use xxx instead` or `is not a function`
- After bumping the library, Copilot still suggests the old API

**Cause**
Copilot's completion is based on training data + currently open files. The library version in training may predate your installed version, and Copilot doesn't read your `package.json`.

**Recovery**
- Open the **new API's docs page as a tab** (Copilot reads open tabs)
- Or annotate the file:

```typescript
// Using TanStack Query v5 API (useQuery signature: useQuery({ queryKey, queryFn }))
import { useQuery } from '@tanstack/react-query';
```

**Prevention**
Declare key library versions and API style in `.github/copilot-instructions.md`:

```markdown
## Key dependency versions
- React 19 (use useTransition, useOptimistic)
- TanStack Query v5 (new useQuery signature)
- Zod v3 (not v2 chained object() style)
```

---

## Pitfall 2: Agent Mode Reads `.env` or Sensitive Files

**Symptom**
- You ask Agent to adjust config; it scans `.env` and reads keys into context
- You see your real API key in the Chat window
- Worst: completion suggestions leak real secrets

**Cause**
Copilot's exclude config isn't on by default. Files in `.gitignore` are still readable. `.env`, `.aws/credentials`, `id_rsa` are all in scope unless explicitly excluded.

**Recovery**
Add excludes to `.vscode/settings.json`:

```json
{
  "github.copilot.enable": {
    "*": true,
    "env": false,
    "secrets": false
  },
  "github.copilot.advanced": {
    "exclude": ["**/.env*", "**/secrets/**", "**/*.pem", "**/*.key"]
  }
}
```

If secrets already hit Chat history: clear Chat (`Ctrl+L` or panel button) **and rotate the affected keys**.

**Prevention**
- Configure excludes at project init, don't wait for an incident
- Enterprise: use GitHub's Copilot Content Exclusion as org-level backstop
- Inject sensitive config via `direnv` / `1Password CLI` from outside the project dir

---

## Pitfall 3: `.github/copilot-instructions.md` Too Long, Silently Ignored

**Symptom**
- You wrote 20 rules in instructions
- Agent follows the first few, ignores the rest
- Especially in Chat, detail rules barely take effect

**Cause**
copilot-instructions.md effectiveness drops sharply past ~150 lines. Copilot truncates on context injection; trailing content gets cut.

**Recovery**
Split by role:
- Stable core rules → `.github/copilot-instructions.md` (< 100 lines)
- Scenario rules → `.github/chatModes/*.chatmode.md`
- Specialist personas → `.github/agents/*.agent.md`

**Prevention**
Keep instructions **cross-scenario globals only** (tech stack, naming, forbidden patterns). Scenario-specific rules:

```
.github/
├── copilot-instructions.md     # Core rules
├── chatModes/
│   ├── security-review.md      # Security review
│   └── write-tests.md          # Test writing
└── agents/
    └── migration-helper.md     # Project migration
```

---

## Pitfall 4: `#file` References Don't Match `@workspace` Finds

**Symptom**
- You say `#file:src/api/user.ts apply this change`
- Copilot gets it mostly right, but not exactly
- Another time you `@workspace` for user.ts — it mentions 2 user.ts (project has two same-named files)

**Cause**
- `#file` precisely references the path you give
- `@workspace` makes Copilot search the whole workspace
- With duplicate filenames, `@workspace` may pick the wrong one

**Recovery**
Explicit path beats fuzzy index:

```
❌ @workspace fix the register method in user.ts
✅ #file:src/api/v2/user.ts fix the register method
```

**Prevention**
- Avoid duplicate filenames in your project (`user.ts` × 3 is a smell)
- If duplicates must exist, always reference by full path, not filename
- Use `@workspace` for **discovery** ("is there an X?"), not for **targeting** ("change X")

---

## Pitfall 5: MCP Server Configured But Not Working

**Symptom**
- You set `github.copilot.chat.mcpServers` in `.vscode/settings.json`
- Chat doesn't use the MCP tool when it should; takes a different path
- No error, just silently unused

**Cause**
Three common reasons:
1. Startup command wrong (`npx` path, arg order)
2. MCP server starts but Copilot version doesn't support it (needs recent VS Code + Copilot Chat)
3. Server starts OK but tool schema is malformed; Copilot can't recognize it

**Recovery**
```
# In Chat, ask directly
What MCP tools can you call right now? List them.
```

If your configured tools aren't listed → MCP didn't load. Check VS Code Output panel → Copilot Chat for errors.

**Prevention**
- Verify MCP with an official sample server (e.g. `@modelcontextprotocol/server-filesystem`) first
- Then swap to your own server
- Keep VS Code and Copilot extensions on latest

---

## Pitfall 6: Completion Behaves Differently in JetBrains vs VS Code

**Symptom**
- Your team's VS Code users get noticeably better completions than JetBrains users
- Same prompt yields different suggestions
- Shared `.github/copilot-instructions.md` takes full effect in VS Code, partial in JetBrains

**Cause**
- VS Code is Copilot's primary target; new features ship there first
- JetBrains support for instructions, MCP, Agent mode lags behind
- Underlying models may differ (JetBrains sometimes uses older models)

**Recovery**
Either standardize IDE team-wide, or accept JetBrains being a step behind.

**Prevention**
- Make IDE consistency a hard constraint when adopting AI tools
- JetBrains users: put **restated core rules at the top** of instructions — even if mid-doc rules don't load, the first few lines do

---

## Pitfall 7: Free Tier Users Hit Hidden Rate Limits

**Symptom**
- Copilot Free worked fine; one day completion latency spikes
- Chat gets slow, "thinking..." for tens of seconds
- No error, just slow, with dropped quality (like a downgraded model)

**Cause**
Copilot Free has monthly caps and rate limits. Hitting them triggers **silent degradation**, not an explicit error. Enterprise users may hit org-level quotas similarly.

**Recovery**
- GitHub Account → Copilot settings to check usage
- Near the cap: upgrade to Pro / Pro+ / Business, or lean on another tool for the rest of the month

**Prevention**
- Heavy users: pay for Pro ($10/mo) from day 1, don't try to squeeze Free
- Teams: align quota and usage strategy with admins
- Build a "Copilot fallback" plan (switch to Claude Code or Cursor when capped)

---

## Pitfall 8: Custom Chat Mode / Agent Not Discovered

**Symptom**
- You wrote a specialist in `.github/chatModes/security-review.md`
- Type `@security-review` in Chat — Copilot doesn't recognize it
- Or it recognizes it but behaves like plain Chat

**Cause**
- Filename and frontmatter `name` don't match
- Missing frontmatter or missing fields
- VS Code / Copilot version too old to support custom Chat Mode

**Recovery**
Verify required frontmatter:

```markdown
---
name: security-review       # Matches filename
description: Security review using OWASP Top 10
---

# Role content below
```

**Prevention**
- Start from an official sample Chat Mode file, modify from there
- Restart VS Code (not just Copilot) after edits
- `.github/chatModes/*.md` path is fixed — don't put it elsewhere

---

## Contribute a Pitfall

Template: see [claude-code.en.md end](./claude-code.en.md#found-a-new-pitfall).

---

## Related

- [Copilot Full Guide](../copilot/README.en.md)
- [common/security.en.md](../common/security.en.md) — AI coding security risks
- [common/context-management.en.md](../common/context-management.en.md) — Context management
