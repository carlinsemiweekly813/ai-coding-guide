[简体中文](./cursor.md) | **English**

# Cursor Pitfalls

> Cursor is powerful but has its own traps. 8 real-world pitfalls, each with Symptom / Cause / Recovery / Prevention.
>
> Hit a new one? Open an Issue or PR.

---

## Pitfall 1: Composer Goes Rogue — Edits Files You Didn't Touch

**Symptom**
- You Composer-edit one feature; the diff spans 6 files
- Some of those files you never opened; it found them across directories and edited them
- Worst case: it "refactored" something in `src/utils/` and broke an unrelated feature

**Cause**
Composer is agent mode with cross-file edit permission by default. It searches related files and decides "these need changing too." Without explicit boundaries, it defines "related."

**Recovery**
```
Stop. Revert all changes outside [files I specified].
Keep only the changes in @src/pages/order/list.tsx.
```

**Prevention**
Set the scope explicitly in the Composer prompt:

```
Use Composer. Only these two files:
@src/pages/order/list.tsx @src/pages/order/detail.tsx

If other files need changes to complete this, tell me first.
```

Or codify in `.cursor/rules/`:

```markdown
## Composer boundaries
- Unless I @ the file, don't modify it
- src/utils/ and src/types/ are shared code — notify before editing
```

---

## Pitfall 2: Tab Blasts Out a Page of Code, Esc Comes Too Late

**Symptom**
- You just wanted a variable name; Tab dumps 30 lines
- The generated code looks OK but has subtle bugs (hardcoded data, wrong type assumptions)
- "Reflexive Undo" becomes a habit — development slows down

**Cause**
Cursor's Tab completion predicts "the next full chunk," not single-line completion. At info-dense points (function/file start), it'll produce a whole function body.

**Recovery**
- Long completion: press `Esc` to reject
- Accepted something bad: `Cmd+Z` to undo, or `Cmd+K` the selection: "what's wrong with this?"

**Prevention**
- Cursor Settings → Features → shorten completion length (or disable `Auto` and require manual trigger)
- Write **function signature and types first**, then let it complete — fuller signature → better completion
- Comment the intent before waiting for completion:

```typescript
// Count business days between two dates, excluding weekends and CN holidays
function countBusinessDays(start: Date, end: Date): number {
  // Tab completion now has enough to work with
}
```

---

## Pitfall 3: `.cursorrules` Too Long, Rules Silently Dropped

**Symptom**
- You spend half a day writing 300 lines of `.cursorrules`
- In practice, the AI ignores half of them
- New rules feel like "shouting into a void"

**Cause**
Single-file `.cursorrules` beyond ~200 lines gets marginalized. Cursor auto-summarizes long docs and drops trailing detail.

**Recovery**
Split into `.cursor/rules/` (the **directory**, not the single file):

```
.cursor/rules/
├── global.md           # Always-loaded core rules (< 100 lines)
├── react.md            # Loads only for .tsx files
├── api.md              # Loads only under src/api/
└── testing.md          # Loads only for *.test.ts
```

Add frontmatter for conditional loading:

```markdown
---
globs: ["src/components/**/*.tsx"]
---

# React component rules
- Props must be declared with interface
- Components must handle loading and error states
```

**Prevention**
- **Start new projects with `.cursor/rules/` directory**, skip `.cursorrules` entirely
- Keep each rule file under 100 lines
- Context-specific rules go in Notepads, referenced on demand

---

## Pitfall 4: `@file` Reference Doesn't Actually Load

**Symptom**
- You say `@src/types/user.ts fix the API to match this type`
- Generated code uses guessed field names/types, not your real ones
- Looking closely: it never actually read the file

**Cause**
Three common reasons:
1. Path is wrong (case / relative vs absolute)
2. File is huge and got truncated (read only the first N lines)
3. You edited but haven't saved (Cursor reads the disk version)

**Recovery**
```
First confirm: what are the fields of User interface in @src/types/user.ts?
List them so I can verify, then start the refactor.
```

Make the AI recite the file contents first to confirm the reference worked.

**Prevention**
- For big files, `@folder` the directory and let it find the right file — more robust than exact path
- **Save (Cmd+S) before @ referencing** changed files
- For huge files (>1000 lines), reference by line range: `@src/models/big.ts:50-120`

---

## Pitfall 5: Chat vs Composer Confusion Kills Productivity

**Symptom**
- In Chat: you ask "refactor this module" — it just suggests, doesn't edit code
- In Composer: you ask "how does this work?" — it doesn't explain, just starts editing
- You keep switching and always pick wrong

**Cause**
- **Chat (`Cmd+L`)**: Q&A mode, no auto-edits
- **Composer (`Cmd+I`)**: Agent mode, prioritizes code changes
- **Inline (`Cmd+K`)**: inline edit, selection only

Three modes, three hotkeys — easy to mix up early.

**Recovery**
Wrong mode: copy the prompt, close the panel, open the right mode, paste.

**Prevention**
Memorize the hotkey ↔ purpose mapping:

| Intent | Mode | Hotkey |
|--------|------|--------|
| Ask a question, understand code | Chat | `Cmd+L` |
| Edit a selection | Inline | `Cmd+K` |
| Multi-file changes | Composer | `Cmd+I` |

---

## Pitfall 6: Model Switching Breaks Style Consistency

**Symptom**
- You alternate between Claude 3.5 and cursor-small
- Code style in the same project splits: some files strict types, others sloppy `any`
- You don't even notice it's the model switch

**Cause**
Cursor supports multiple models with different default styles:
- Claude Sonnet: strict, adds types and error handling
- GPT-4: terse, minimal comments
- cursor-small: fast, cuts corners on complex tasks

`Auto` mode (system picks per task) hides this drift.

**Recovery**
Push style into `.cursor/rules/global.md` so it applies regardless of model:

```markdown
# Cross-model style
- TypeScript required, no `any`
- Functions must declare return types
- Errors use Result<T, Error>, don't throw
```

**Prevention**
- **One project, one primary model**, locked in settings
- Switch explicitly per task ("use Opus for this deep refactor")
- Don't rely on `auto`

---

## Pitfall 7: Notepads Become Stale Context

**Symptom**
- A Notepad from six months ago has outdated info (old API design, deprecated style)
- The AI references it in a new chat and gives outdated advice
- You don't remember creating that Notepad

**Cause**
Notepads are **manually managed** context. Cursor never auto-updates them. Over months, Notepads drift from real code.

**Recovery**
- Periodically prune: if the AI suggests something weird, suspect Notepad pollution — check the Notepad panel
- Delete or update stale entries immediately

**Prevention**
- Notepads = **stable conventions only** (API response shape, auth flow — rarely changing)
- Volatile things (current feature, temporary plan) → reference live code via `@folder`
- Date your Notepads: review everything older than N months

---

## Pitfall 8: Stale Index — @ References Deleted Files

**Symptom**
- You renamed or deleted a file
- Chat still `@`-references it with old contents
- AI suggests changes based on the old file; you apply them; compile fails ("file not found")

**Cause**
Cursor's codebase index doesn't update in real time. After deletions/renames, the in-memory index holds old entries for a while.

**Recovery**
Command Palette (`Cmd+Shift+P`) → `Cursor: Resync Index`

**Prevention**
- After large renames/deletions, resync manually
- If you branch-switch a lot, resync after each switch
- If `@` references behave weirdly, check the index first — don't blame the AI

---

## Contribute a Pitfall

Template: see [claude-code.en.md end](./claude-code.en.md#found-a-new-pitfall).

---

## Related

- [Cursor Full Guide](../cursor/README.en.md)
- [common/context-management.en.md](../common/context-management.en.md) — Cross-tool context management
- [workflows/scenarios.en.md](../workflows/scenarios.en.md) — Includes Cursor + Claude Code scenarios
