---
name: claudify
description: "Audit recurring workflows, rituals, and processes and decide whether to automate each with Claude, kill it, reshape it, or deliberately keep it human. Covers two kinds of ritual: the ones people perform (standups, planning, code review, onboarding, reporting, triage, re-typing the same prompts) and the ones Claude itself re-performs every session in a repo (pull latest, push at end, rewrite handoff docs, re-derive shell gotchas, remember prose rules that a hook could enforce). Use when the user asks 'what should I automate / claudify', 'is this process still worth it', 'should we still do X', 'we keep doing Y every week, is it needed', 'audit my workflow / our rituals', 'where can Claude help in how we work', 'go through my repos and find what to claudify', 'what should become a hook or a skill', or Czech 'co mám zautomatizovat', 'audit našich rituálů', 'co by mělo být hook'. Also use proactively when the user describes a process that sounds like it survives only out of habit, or when a memory/CLAUDE.md rule is being re-explained for the second time. Produces a ranked verdict table (claudify / kill / reshape / keep-human) with a first concrete step, a 'Start here' for the human and one for Claude, and a hooks-to-add list. Based on the 'Running an AI-native engineering org' talk (Fiona Fung). Do NOT use for one-off tasks; this is for *recurring* processes."
---

# Claudify

A repeatable audit for recurring processes, distilled from Fiona Fung's "Running an
AI-native engineering org". The premise: for years a scarce resource (engineering
bandwidth, analyst time, your own hours) was expensive, so processes were built to ration
it. When the cost of producing a first draft collapses, many of those processes quietly
stop earning their place, but nobody deletes them, they just pile new ones on top. This
skill names the ones that can go.

## The core question
> If this only exists because [some resource] used to be expensive, does it still earn
> its place now that Claude can do the first pass?

## Two audit targets
1. **Rituals the human performs**: meetings, reports, reviews, hand-pasted prompts,
   copy-based backups, filename versioning, manual monitoring.
2. **Rituals Claude performs every session** in the user's repos: pull latest, push at
   end, rewrite the handoff/state document, re-derive environment gotchas, remember prose
   rules from CLAUDE.md or memory. A rule a model must re-read and re-execute each session
   is a ritual too, and usually the cheapest fix is a hook, not better prose.

## When to use / not
- Use: auditing *recurring* workflows/rituals, for a team, a person, or a whole workspace
  of repos.
- Don't use: one-off tasks, or building the automation itself (that's a skill/hook/plugin
  job; see customization-principles.md for how to pick the abstraction).

## Step 1 — list the workflows
If the user gave a list, use it. Otherwise elicit: "What do you and your team do on a
repeating cadence: daily, weekly, per-PR, per-cycle?" Capture each as a one-liner.

For a **whole workspace** (many repos), don't read serially. Fan out readers, one per repo
group, plus cross-cutting miners: every project memory dir, global tooling (skills, hooks,
settings), remote repos via `gh api`, git-log cadence (commit-prefix histograms, checkpoint
and handoff commits), and session transcripts (most-repeated short user asks; aggregates
only, never dump user text). Merge into one deduplicated list; a ritual seen in 2+
projects outranks a single-repo one. Then verify in **batches** (5-8 items per verifier
agent, all lenses at once), not one agent per item per lens, or the token bill explodes.
Skeptic agents under-refute: spot-check the boldest claims by hand before handing over.

## Step 2 — score each workflow
For every workflow, answer five things briefly:
1. **Noise**: how often it runs and how painful/slow it is (this sets priority).
2. **Origin**: why it exists. Flag any "because [X] used to be expensive".
3. **Bottleneck now**: what's actually slow today (often *not* the original constraint;
   it has usually moved to verification, review, or decision-making).
4. **Existing tooling**: which installed skill, hook, script, or workflow already covers
   part of it. The most common finding is a wiring gap, not missing authorship: the skill
   exists and nothing triggers it. Prefer extending over building.
5. **Verdict** (one of):
   - **Claudify**: Claude can do the first draft or the whole loop. Name the cheapest
     abstraction: hook > skill > script > agent/workflow > MCP. A prose rule is not the
     cheapest abstraction when a hook can enforce it.
   - **Kill**: theater; the artifact isn't load-bearing anymore.
   - **Reshape**: keep the goal, change the form (e.g. "design doc *before* code" becomes
     "prototype first, doc after if it needs to exist"; "paste four prompts by hand"
     becomes "one project skill").
   - **Keep-human**: judgment, taste, trust, risk, money, credentials, or sending things to
     third parties live here; protect it (see below).

## What Claude should NOT own (keep-human)
From the deck's review split: Claude handles style/lint, obvious bugs, spec drift and
missing tests, repeated patterns, triage. Humans keep **risk tolerance, product sense and
taste, trust boundaries, security-sensitive code**, and the final decision. Humans stop
being the first draft; they don't stop being the decider. Concretely: money and invoices,
credentials and OAuth grants, deletes, publish go/no-go, ratifying defaults.

## Norm prompts (don't miss these six)
Run the user's processes against the norms the deck rebuilt; each is a likely candidate:
- **Code/work review**: apply human judgment only where it matters; automate the rest.
- **Onboarding**: the cost of a "dumb" question went to ~0; ramp should be faster.
- **Planning**: less upfront, more prototype. In technical debates, *code wins*: build
  both options and compare the artifact instead of arguing.
- **Hiring / make-up**: index on judgment, taste, and what to build, not raw output.
- **Org shape / ownership**: flatter; "who wrote this" is a weirder question now.
- **Session boundaries** (Claude-side): handoff and state docs that get rewritten whole
  every session grow without bound. Append dated deltas from a SessionEnd hook, surface the
  last delta from a SessionStart hook, consolidate when the file crosses ~20 KB.
Plus the one thing to double down on, not cut: **verification**. Shift it left and keep
automating it, because AI-native work breaks in new ways.

## Step 3 — output
Write `claudify_<topic>.md` to the workspace root with:
- **Start here (human)**: the single noisiest ritual the person does that no longer earns
  its place, with the first step. One thing at a time.
- **Start here (Claude)**: the single noisiest thing Claude re-does every session that
  should become a hook, skill, or script.
- a ranked table: *workflow · who · cadence · noise · verdict · cheapest abstraction ·
  first step*, sorted by noise; mark keep-human overrides in the verdict cell;
- **Kill list** (framed as proposals, with a diff-before-delete guard), **Reshape**,
  **Keep-human**;
- **Hooks to add**: event, matcher, command sketch for each prose rule a hook replaces;
- **Existing tooling to extend instead of building new**;
- **Refuted / dropped**, so the reader sees what was considered;
- three numbers to baseline now and re-check in ~6 months, e.g. hooks configured,
  state-doc size, duplicate-artifact count (or the deck's: onboarding ramp time,
  end-to-end cycle time, Claude-assisted share of output).

Keep alignment in mind: surfacing that a ritual is theater is a people conversation, not
just a verdict. Frame kills as proposals, not decrees, and execute them as reversible
moves (archive folder, `gh repo archive`) rather than deletes.
