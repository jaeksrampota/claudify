# Customization principles — building skills/plugins/hooks

Heuristics to apply whenever you build a skill, plugin, hook, or agent.
Distilled from the Code with Claude talk "Beyond the basics" (Daisy Hollman), plus
field notes from applying them.

## Pick the abstraction by prompt-cost — there is a clear order
Cheapest-on-prompt first: **hooks > skills > agents >> MCP**.
- **Hooks** (shell commands on PreToolUse / PostToolUse / UserPromptSubmit / Stop /
  SessionStart / SessionEnd): the hook itself adds nothing to the prompt — it runs a local
  command when the event fires, and you pay tokens only for whatever output you feed back
  into the conversation. Best for tight feedback loops ("lint the diff and feed the warning
  back", "you edited a generated file — run the generator") and for **replacing prose rules**
  ("always pull first", "write .cmd files as ASCII") that a model would otherwise have to
  remember every session.
- **Skills**: just files. The one-line description is always loaded; the full SKILL.md +
  assets load only when invoked. Default choice for a procedure.
- **Agents / subagents**: the work happens off-box (an agent can read 50 files so the main
  loop doesn't), but each agent's description still sits in the parent prompt and
  coordination cost grows with fan-out.
- **MCP**: every tool's name + schema sits in the system prompt (tool-search shrinks the
  size cost, not the auth/process-lifecycle work). Reach for MCP only when there is **no
  CLI to drive** or you need cross-client portability. If a CLI already exists, a skill
  that drives it is less code and fewer moving parts.

## Skill authoring
- The description is **always loaded**, so make triggering reliable: write a *paragraph*
  of "use when the user says X, or Y, or asks about Z…", not a one-liner.
- Keep the body pay-per-use: heavy procedure + scripts/fixtures live in the body and load
  only on invocation. Don't pay for what you don't use.
- Skills are just files — version, review, share like code.
- **Check existing skills first.** Before building, diff the idea against the installed
  skill list. The usual gap is that a skill exists and nothing triggers it; wire it (a hook,
  a pointer in CLAUDE.md, a launcher script) instead of writing a sibling.

## Hooks in practice (field notes, Windows / PowerShell 5.1)
- Run PowerShell explicitly: `powershell.exe -NoProfile -NonInteractive -ExecutionPolicy
  Bypass -File C:/path/hook.ps1`. The hook option `shell: "powershell"` means `pwsh`
  (PowerShell 7), which a 5.1-only machine does not have. Forward slashes in the path work
  from both Git Bash and cmd.
- Hooks receive JSON on stdin (`[Console]::In.ReadToEnd() | ConvertFrom-Json`). Test with
  JSON produced by a real serializer (`python -c "import json; ..."`), not a shell `echo`
  of hand-typed backslashes — the shell will eat the escapes and you will debug a bug that
  does not exist.
- Spawning an interpreter on every tool call costs ~0.3 s each. Put a cheap shell
  pre-filter in the hook command and only spawn PowerShell when the payload matters:
  `IN=$(cat); case "$IN" in *git*push*) printf %s "$IN" | powershell.exe ...; exit $?;; esac; exit 0`.
  The `if` filter with mid-string wildcards (`Bash(*git push*)`) did not fire in testing;
  don't rely on it for anything that must not be missed.
- Output channels: SessionStart stdout becomes model context (use it for an orientation
  digest). PostToolUse can return `{"hookSpecificOutput":{"hookEventName":"PostToolUse",
  "additionalContext":"..."}}` for information and exit 2 with stderr only for a real
  problem. PreToolUse exit 2 blocks the tool and shows stderr to the model.
- SessionEnd is the place for mechanical handoff deltas (commits since last entry,
  uncommitted files). Write them **outside the repo** (e.g. `~/.claude/session-logs/`) so
  the hook never dirties the tree, and print the last entry from SessionStart.
- Anything destructive (auto-push, deletes) gets gates the script cannot talk itself past:
  clean tracked tree, rebase succeeds, tests pass, and a log line for every outcome.

## Context is a box; you are the package manager
- Everything competes for one window. "Just include it" is almost never fine —
  "don't pay for what you don't use" is the whole game, not a nice-to-have.
- **KV-cache ordering**: stable/shared content up front, volatile/per-task content last.
  Change one byte near the front and everything after it recomputes — so it is *not* an
  LRU problem; the cache key is the prefix.
- Fan-outs are not free either: batch verification (5-8 items per agent), cap agents per
  workflow, and reserve the expensive model for the single hardest synthesis step.

## Be AGI-pilled
Two kinds of tooling: those that **compensate for lack of intelligence** (rigid templates,
guardrails, "don't let it touch X" — decay as models improve) vs those that **scale with
intelligence** (more access, more context, faster feedback loops — grow as models
improve). Build mostly the second kind. Build for the next model, not the last one.

## Memory / CLAUDE.md discipline
You pay for all of it every turn, even the ~90% irrelevant to the task, and it doesn't
compose. Keep memory lean — only the handful of rules/facts that truly apply everywhere.
Push procedures into skills (pay-per-use), not into memory; push event-shaped rules into
hooks (zero prompt cost) and leave a one-line pointer.

## Access first
If Claude can't reach it, Claude can't help with it. Before building cleverness, check
whether the blocker is just missing access/context — a connector, a file, a CLI.
