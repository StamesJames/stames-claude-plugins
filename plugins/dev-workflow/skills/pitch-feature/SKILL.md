---
name: pitch-feature
description: Pick the single item from CLAUDE_PLAN.md that would be best to implement right now, explain to the user what it is and how it benefits the project, and implement it only if the user approves. Never implements more than that one item. Use when the user asks what Claude would build next, for a feature pitch or recommendation from Claude's plan, or to implement something from CLAUDE_PLAN.md. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Pitch one feature from Claude's plan

`CLAUDE_PLAN.md` (repo root) holds Claude's own ideas, written by `/plan-features`. This skill
pitches **one** item, and builds it only after the user explicitly approves (building them
all at once is `/implement-all-claude-features`, which asks for confirmation first). Never loop, never chain a second item in the same
run, and never run this skill unattended or in a subagent.

Optional argument: an item id (e.g. `F3`) to pitch that one, or a focus (e.g. `quick`,
`onboarding`) to pick within it.

## 1. Check for open points

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`,
   `CLAUDE.md`, `USER_WORKFLOW.md` and `CLAUDE_PLAN.md`. If the plan holds no items, say so,
   suggest `/plan-features`, and stop.
3. Read the **Design docs** named in `USER_WORKFLOW.md`, `USER_PLAN.md` and
   `git log --oneline -15`, so you know the intent and what the user is working on right now.

## 2. Pick the best item for *now*

Not simply the highest severity. Weigh:

- **Concrete features** over ideas (an idea can be pitched, but then the pitch is about
  settling its open questions, and implementing comes after).
- Value for the project at its current stage: what its users miss most right now.
- Fit with recent work and with `USER_PLAN.md` (builds on what was just made, unblocks or
  complements the user's own items; doesn't collide with something the user is doing).
- Effort and risk: a smaller item with nearly the same value wins.
- `[CONFLICT]` items only if they are clearly the best; the pitch must then say so up front.
- Never items tagged **[HOLD]**, not even when named, unless the user lifts the hold.

**Check the item against the current code** before pitching: if it is already done, delete
the entry, say so and pick another one. If the code moved on, adjust the plan for the pitch.

## 3. Pitch it

Tell the user, short and in plain words:

- **What** it is, as the user of the product experiences it.
- **Why now**: how it benefits the project and why this one beats the other open items.
- **How**: the approach, the files/systems touched, the effort, and the risks (persisted
  formats, behaviour changes, load-bearing names, anything under **Ask before**).
- **Conflict**: for a `[CONFLICT]` item, what in the design docs it contradicts and why it's
  worth it anyway.
- **Decisions** the user has to make, if any, with your recommendation.

Then ask with `AskUserQuestion` (put the decisions in the same call if there are any):

- `Implement it` (recommended) — build it as pitched.
- `Pitch another` — pick the next best item and pitch that one instead (once; if that is
  turned down too, stop).
- `Not now` — keep the item in the list, stop.
- `Drop it` — remove the entry from its section and add one line to `## Dropped` at the end
  of `CLAUDE_PLAN.md`: `- <id> <name> — <the user's reason> (<date>)`. If the user gave no
  reason, ask for one in a single short question (they may answer "none"). Then stop.

Anything else (the user's own text) is a change to the plan: adapt the pitch and ask again.
No project code is edited before an explicit `Implement it`.

## 4. Implement it (only after approval)

Follow **steps 3–5 and 7 of `${CLAUDE_PLUGIN_ROOT}/skills/implement-next-feature/SKILL.md`** (implement,
verify, update the plan file, commit), with these substitutions:

- The item comes from `CLAUDE_PLAN.md`; that is the file whose entry is deleted (or rewritten
  with an `_Updated:_` stamp if only partly done), along with its mentions in other items.
- Follow the approach and decisions the user approved. If you hit something that changes
  them, stop and ask again instead of deviating silently.
- For a `[CONFLICT]` item, also tell the user which part of the design docs is now out of date
  (don't edit them unless they ask).
- Commit footer: `Refs: claude plan <id> (<title>)`.
- Attended only: no `RESULT:`/`BLOCKED:`/`SCOPE:` lines; mention a missing scope in the report.

## 5. Report back

In a few short sentences: what you built, the files changed, how you verified it and what the
user should check by hand, the commit (with a scope suggestion if no scope fit), and how many
items are left in `CLAUDE_PLAN.md`. Don't offer to start the next one right away; the user runs
`/pitch-feature` again when they want one.
