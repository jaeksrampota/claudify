---
name: claudify
description: "Audit a recurring workflow, ritual, or process and decide whether to automate it with Claude, kill it, reshape it, or deliberately keep it human. Use when the user asks 'what should I automate / claudify', 'is this process still worth it', 'should we still do X', 'we keep doing Y every week — is it needed', 'audit my workflow / our rituals', 'where can Claude help in how we work', or wants to review standups, planning, code review, onboarding, reporting, triage, or any repeated chore. Also use proactively when the user describes a process that sounds like it survives only out of habit. Produces a ranked verdict table (claudify / kill / reshape / keep-human) with a first concrete step, and names the single noisiest thing to start on. Based on the 'Running an AI-native engineering org' talk (Fiona Fung). Do NOT use for one-off tasks — this is for *recurring* processes."
---

# Claudify

A repeatable audit for recurring processes, distilled from Fiona Fung's "Running an
AI-native engineering org". The premise: for years a scarce resource (engineering
bandwidth, analyst time, your own hours) was expensive, so processes were built to ration
it. When the cost of producing a first draft collapses, many of those processes quietly
stop earning their place — but nobody deletes them, they just pile new ones on top. This
skill names the ones that can go.

## The core question
> If this only exists because [some resource] used to be expensive, does it still earn
> its place now that Claude can do the first pass?

## When to use / not
- Use: auditing *recurring* workflows/rituals — standups, planning, design-doc rituals,
  code review, onboarding, status reports, triage, weekly bundles.
- Don't use: one-off tasks, or building the automation itself (that's a skill/plugin job —
  see customization-principles.md for how).

## Step 1 — list the workflows
If the user gave a list, use it. Otherwise elicit: "What do you and your team do on a
repeating cadence — daily, weekly, per-PR, per-cycle?" Capture each as a one-liner.

## Step 2 — score each workflow
For every workflow, answer four things briefly:
1. **Noise** — how often it runs and how painful/slow it is (this sets priority).
2. **Origin** — why it exists. Flag any "because [X] used to be expensive".
3. **Bottleneck now** — what's actually slow today (often *not* the original constraint;
   the bottleneck has usually moved to verification / review / decision-making).
4. **Verdict** (one of):
   - **Claudify** — Claude can do the first draft / the whole loop. Note the cheapest
     abstraction (skill, hook, or just a reusable prompt; reach for MCP last).
   - **Kill** — it was theater; the artifact isn't load-bearing anymore.
   - **Reshape** — keep the goal, change the form (e.g. "design doc *before* code" →
     "prototype first, doc after if it needs to exist").
   - **Keep-human** — judgment, taste, trust, or risk lives here; protect it (see below).

## What Claude should NOT own (keep-human)
From the deck's review split — Claude handles style/lint, obvious bugs, spec drift &
missing tests, repeated patterns, triage. Humans keep: **risk tolerance, product sense &
taste, trust boundaries, security-sensitive code**, and the final decision. Humans stop
being the first draft; they don't stop being the decider.

## Norm prompts (don't miss these five)
Run the user's processes against the five norms the deck rebuilt — each is a likely
candidate:
- **Code/work review** — apply human judgment only where it matters; automate the rest.
- **Onboarding** — the cost of a "dumb" question went to ~0; ramp should be faster.
- **Planning** — less upfront, more prototype. In technical debates, *code wins* — build
  both options and compare the artifact instead of arguing.
- **Hiring / make-up** — index on judgment, taste, and what to build, not raw output.
- **Org shape / ownership** — flatter; "who wrote this" is a weirder question now.
Plus the one thing to double down on, not cut: **verification** — shift it left and keep
automating it, because AI-native work breaks in new ways.

## Step 3 — output
Write `claudify_<topic>.md` to the workspace root with:
- a ranked table: *workflow · noise · verdict · first step*, sorted by noise;
- **"Start here"** — the single noisiest workflow that doesn't earn its place. One thing
  at a time; don't boil the ocean.
- optionally, three numbers to baseline now and re-check in ~6 months: onboarding ramp
  time, end-to-end cycle time, and Claude-assisted share of output.

Keep alignment in mind: surfacing that a ritual is theater is a people conversation, not
just a verdict — frame kills as proposals, not decrees.
