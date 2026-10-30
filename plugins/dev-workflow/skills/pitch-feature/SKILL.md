---
name: pitch-feature
description: Pick the single concrete feature from CLAUDE_PLAN.md that would be best right now, explain what it is and how it benefits the project, and on the user's answer build it now, approve it into USER_PLAN.md for the next run, or drop it. Never builds more than that one item. Use when the user asks what Claude would build next, for a feature pitch or recommendation from Claude's plan. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Pitch one feature from Claude's plan

`CLAUDE_PLAN.md` (repo root) holds Claude's own suggestions, written by `/plan-features`.
Nothing in it is built without the user's approval. This skill pitches **one** concrete
feature; the user then has it built right away, approves it into `USER_PLAN.md` (so the next
`/implement-all-features` run builds it), or drops it. Never loop, never chain a second build
in the same run, and never run this skill unattended or in a subagent.

Optional argument: an item id (`F3`) to pitch that one, or a focus (`quick`, `onboarding`) to
pick within it. Ideas (`I` items) are not pitched: for one, suggest
`/explore-feature claude:<id>`.

## 1. Check for open points

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`, `CLAUDE.md`,
   `USER_WORKFLOW.md` and `CLAUDE_PLAN.md`. If it holds no concrete features, say so, suggest
   `/plan-features` (or `/explore-feature` for its ideas), and stop.
3. Read the design-doc sections the candidates name, `USER_PLAN.md` and
   `git log --oneline -15`, so you know the intent and what the user is working on right now.

## 2. Pick the best item for *now*

Not simply the highest severity. Weigh:

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
- **How**: the approach, the files/systems touched, the size, and the risks (persisted
  formats, behaviour changes, load-bearing names, anything under **Ask before**).
- **Conflict**: for a `[CONFLICT]` item, what in the design docs it contradicts and why it's
  worth it anyway.
- **Decisions** the user has to make, if any, with your recommendation.

Then ask with `AskUserQuestion` (put the decisions in the same call if there are any):

- `Implement it now` (recommended for a small item) — build it as pitched, in this session.
- `Approve for later` (recommended for a larger one) — move it to `USER_PLAN.md` (step 5).
- `Pitch another` — pitch the next best item instead (once; if that is turned down too, stop).
- `Drop it` — remove the entry and add one line to `## Dropped` at the end of
  `CLAUDE_PLAN.md`: `- <id> <name> — <the user's reason> (<date>)`. If the user gave no
  reason, ask for one in a single short question (they may answer "none"). Then stop.

"Not now" or any other answer (the user's own text) keeps the item and stops, or, if it
changes the plan, adapt the pitch and ask again. No project code is edited before an explicit
`Implement it now`.

## 4. Implement it now (only after approval)

Follow **sections C to G of `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/work-item.md`**
(build, self-review, verify, update the list, commit), attended, with these specifics:

- The item comes from `CLAUDE_PLAN.md`; that is the file whose entry is deleted (or rewritten
  with an `_Updated:_` stamp if only partly done), along with its mentions in other items.
- Follow the approach and decisions the user approved. If you hit something that changes
  them, stop and ask again instead of deviating silently.
- For a `[CONFLICT]` item, also tell the user which part of the design docs is now out of date
  (don't edit them unless they ask).
- Commit type usually `feat`; footer `Refs: claude plan <id> (<title>)`.

## 5. Approve for later

Move the entry to `USER_PLAN.md` → `## Concrete features` with a new id from its `_Next id:_`
line and a fresh `_Written:_` stamp. Keep its text, add a **Done when:** if it has none
(derived from **What**, shown to the user in the report), and add the answers to the pitch's
decisions as **Decisions:**. Delete it from `CLAUDE_PLAN.md` and update references to the old
id (**Keep open for:** lines, other items) in both plans. Commit both plan files (type
`chore`, e.g. `chore: approve pause menu sound for the plan (F9)`, no `Refs:` footer).

## 6. Report back

In a few short sentences: what you built (files, how you verified it, what the user should
check by hand, the commit with a scope suggestion if no scope fit), or the new `USER_PLAN.md`
id, and how many items are left in `CLAUDE_PLAN.md`. Don't offer to start the next one right
away.
