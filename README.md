# claudify

A [Claude Code](https://claude.com/claude-code) **skill** that audits your *recurring*
workflows and rituals — standups, planning, code review, onboarding, status reports,
triage, re-typed prompts — and gives each a verdict: **claudify** (let Claude do the first
pass), **kill** (it's theater), **reshape** (keep the goal, change the form), or
**keep-human** (judgment, taste, trust, and risk live there).

It audits two kinds of ritual: the ones *you* perform, and the ones *Claude* re-performs
every session in your repos (pull latest, push at end, rewrite the handoff doc, remember
prose rules from CLAUDE.md that a hook could enforce). Prose a model must re-read each
session is a ritual too, and the cheapest fix is usually a hook.

Distilled from Fiona Fung's *"Running an AI-native engineering org"* talk. The core question:

> If this only exists because some resource used to be expensive, does it still earn its
> place now that Claude can do the first pass?

## Install
```sh
cp -r .claude/skills/claudify /path/to/project/.claude/skills/
```
or into your user-level `~/.claude/skills/`. Claude Code auto-discovers it from `SKILL.md`.

## Output
A ranked `claudify_<topic>.md` — *workflow · who · cadence · noise · verdict · cheapest
abstraction · first step*, sorted by noise — with a **"Start here"** for you and one for
Claude, a kill list framed as proposals, a **hooks-to-add** list (event, matcher, command
sketch for each prose rule a hook replaces), the existing tooling to extend instead of
building new, and three numbers to re-check in six months.

## Worked example (one workspace, 2026-09)
38 local folders, 32 GitHub repos, every project memory dir, git history and session
transcripts, read by 16 parallel agents and merged. Result: 36 recurring rituals, 21
claudify, 13 reshape, 1 kill, 1 keep-human. The single biggest finding was structural:
zero hooks configured on the machine while a dozen prose rules were being re-read every
session. Five hooks and one weekly health-check task later, the rules are enforced instead
of remembered. `customization-principles.md` carries the field notes from building them.

## Files
- `.claude/skills/claudify/SKILL.md` — the audit procedure.
- `.claude/skills/claudify/customization-principles.md` — how to pick the abstraction
  (hooks > skills > agents >> MCP) and hook-building notes for Windows / PowerShell 5.1.
