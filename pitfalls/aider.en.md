[简体中文](./aider.md) | **English**

# Aider Pitfalls

> Aider's pitfalls differ from IDE-based tools: auto-commit, manual `/add /drop`, multi-model switching — every mechanism can bite. 7 real traps.
>
> Hit a new one? Open an Issue or PR.

---

## Pitfall 1: Auto-Commit Swallows Your Working Tree Changes

**Symptom**
- You have uncommitted changes across several files
- You ask Aider to modify an unrelated file
- After generating code, it auto-commits — and **your unstaged changes get bundled into that commit**

**Cause**
Aider defaults to `auto-commits: true`. Its commit scope is the entire working tree diff — it doesn't distinguish "yours" from "mine." It assumes "change = commit."

**Recovery**
```bash
git reset HEAD~1        # Undo last commit, files back to working tree
git status              # Separate yours from Aider's
# Manually commit what should be committed, discard the rest
```

**Prevention**
Pick one:
1. **`git stash` before the session**: hide your WIP so Aider's commit stays clean
2. **Disable auto-commit**: `aider --no-auto-commits`, or `auto-commits: false` in `.aider.conf.yml`
3. **Dedicated branch**: `git checkout -b feature/xxx` before launching Aider

---

## Pitfall 2: `/add` Misses a File, AI Hallucinates to Fill the Gap

**Symptom**
- You ask Aider to change `ServiceA`, it does
- But ServiceA depends on `UtilB` whose signature actually changed — Aider doesn't know, generates code calling a nonexistent signature
- Runtime: `TypeError`

**Cause**
Aider only reads files that are explicitly `/add`-ed. Map mode indexes structure (names + positions) but **not contents**. Dependencies not added → AI guesses from names.

**Recovery**
```
/add src/utils/UtilB.ts
# Let it re-read with the dependency included
Re-check ServiceA's use of UtilB; make sure you're using the right signature
```

**Prevention**
Have Aider enumerate dependencies first:

```
/ask
I'm about to refactor ServiceA. List all files it directly depends on,
and all files that call its public API. Give me /add commands
so I can add them all at once.
```

Then `/add` everything it listed before starting `/code`.

---

## Pitfall 3: Model Switched, Behavior Collapsed

**Symptom**
- You used Claude, got used to its rigor
- Switched to DeepSeek to save money; **same prompts, quality drops a cliff**
- Complex refactors start producing odd bugs; Architect plans become shallow

**Cause**
LLMs vary wildly in instruction-following. Same `/architect design a WebSocket notification system`:
- Claude Sonnet: asks follow-ups, handles error paths
- DeepSeek: gives one plan, skips edge cases
- Local 7B: structurally correct but detail-poor

**Recovery**
- Complex task, model can't keep up: switch back to Claude for that step only, then switch back
- Configure tiered models: strong for reasoning, weak for commit-messages/simple tasks

**Prevention**
Tier in `.aider.conf.yml`:

```yaml
model: claude-3-5-sonnet           # Primary
weak-model: deepseek/deepseek-chat # Weak tasks (commit messages, simple completion)
editor-model: claude-3-5-haiku     # Editor-mode completion
```

Or switch **by task type**:

| Task | Model |
|------|-------|
| `/architect` design | Claude Sonnet/Opus |
| `/code` implement an approved plan | DeepSeek is fine |
| `/ask` Q&A | any |
| Large refactor | Claude Opus |

---

## Pitfall 4: Lint/Test Auto-Loop Blows Up the Bill

**Symptom**
- You enabled `auto-lint: true` and `auto-test: true`
- Aider edits, lint fails, it retries the fix, fails, retries...
- A single session burns 5× expected tokens

**Cause**
Aider's auto-lint/test retries "if it fails, let the LLM try again." If the problem isn't code-fixable (environment, dependencies), it'll retry until it hits the retry cap. Each retry costs tokens.

**Recovery**
```
/undo                     # Roll back this round
/lint-cmd none            # Temporarily disable auto-lint
# Fix the real problem (environment/config), then re-enable
```

**Prevention**
- Cap retries: `aider --max-reflections 3`
- Use auto-fixing linters so the linter resolves what it can before handing to AI:

```yaml
lint-cmd: "eslint --fix"   # not "eslint"
```

- Fail-fast tests:

```yaml
test-cmd: "pytest -x --maxfail=1"
```

---

## Pitfall 5: Stale Map — References Deleted Files

**Symptom**
- Project structure changed (directories removed, modules renamed)
- Aider still "remembers" the old structure, generates imports to nonexistent modules
- Or `/ls` output doesn't match disk

**Cause**
Map mode is built **at session start**. Large git-level changes (branch switch, rebase, mass deletions) during a session don't auto-refresh the map.

**Recovery**
Exit and relaunch Aider — the map rebuilds.
Or in-session:
```
/reset             # Clear session context
/map-refresh       # Re-scan project structure (if your version supports it)
```

**Prevention**
- Restart Aider after branch switches (don't span branches in one session)
- Restart after mass file changes
- Long sessions: `/reset` periodically to refresh state

---

## Pitfall 6: Architect Plan Is Solid, Code Execution Drifts

**Symptom**
- `/architect` produces a clean plan, you approve it
- Switch to `/code`, first couple files match — then it drifts
- Final implementation looks almost nothing like the plan

**Cause**
Architect and Code modes **share session context**, but the Architect plan is long; as Code mode edits file after file, early planning content gets pushed out of context.

**Recovery**
```
# After each implementation step, checkpoint against the plan
/ask Compare the current implementation to the Architect plan.
Where has it drifted?
```

**Prevention**
- Immediately write the Architect plan to a file: `Save this plan to docs/arch.md. We'll implement from that file.`
- At implementation time, re-reference: `/add docs/arch.md Implement the next step per this file.`
- For complex tasks, **one session per sub-module** — avoid long-session drift

---

## Pitfall 7: `--amend` Goes Sideways After Aider Commits

**Symptom**
- You ask Aider to adjust something (auto-committed)
- It makes another commit
- You want to "fold these together," so you `--amend`
- But amend sucks in your **previously stashed content**, or Aider's two changes into one — history ends up messier

**Cause**
Aider commits "after each change," not atomically per logical unit. Manual amend pulls in all currently-staged content. Aider itself doesn't support fine-grained commit control.

**Recovery**
```bash
git reflog                    # Find pre-amend state
git reset --hard HEAD@{N}     # Roll back
# Rewrite history cleanly
git rebase -i HEAD~5
```

**Prevention**
- Squash at end of session:
  ```bash
  git reset --soft HEAD~N   # N = number of Aider commits
  git commit -m "feat: add notifications"
  ```
- Don't `--amend` during an Aider session
- Tag key checkpoints yourself: `git tag wip-before-aider` — rollback is cheap

---

## Contribute a Pitfall

Template: see [claude-code.en.md end](./claude-code.en.md#found-a-new-pitfall).

---

## Related

- [Aider Full Guide](../aider/README.en.md)
- [common/task-decomposition.en.md](../common/task-decomposition.en.md) — Task decomposition (Aider relies on this heavily)
- [common/debugging.en.md](../common/debugging.en.md) — Systematic debugging
