# claudify

A [Claude Code](https://claude.com/claude-code) **skill** that audits your *recurring*
workflows and rituals — standups, planning, code review, onboarding, status reports,
triage — and gives each a verdict: **claudify** (let Claude do the first pass), **kill**
(it's theater), **reshape** (keep the goal, change the form), or **keep-human** (judgment,
taste, trust, and risk live there).

Distilled from Fiona Fung's *"Running an AI-native engineering org"* talk. The core question:

> If this only exists because some resource used to be expensive, does it still earn its
> place now that Claude can do the first pass?

## Install
```sh
cp -r .claude/skills/claudify /path/to/project/.claude/skills/
```
or into your user-level `~/.claude/skills/`. Claude Code auto-discovers it from `SKILL.md`.

## Output
A ranked `claudify_<topic>.md` — *workflow · noise · verdict · first step*, sorted by
noise, with a single "Start here" pick (the noisiest ritual that no longer earns its place).
