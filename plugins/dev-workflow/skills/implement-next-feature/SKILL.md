---
name: implement-next-feature
description: Implement one concrete feature from CLAUDE_PLAN.md (the best next one, or a given id), verify it, delete its entry and commit. Never ideas (USER_IDEAS.md, CLAUDE_IDEAS.md). Use when the user asks to implement the next feature from the plan or names a CLAUDE_PLAN.md feature to build. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Implement the next planned feature

`CLAUDE_PLAN.md` (repo root) is the list of features to build, planned by `/plan-features`
from the user's ideas and approved by the user. This skill implements **one concrete
feature**: pick it, build it, verify it, delete its entry and commit.

Before anything else:

- If `USER_WORKFLOW.md` is missing, the project isn't set up for this workflow: tell the user
  to run `/setup-workflow` and stop.
- Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/work-item.md`, then the context its
  section A lists (part 1 of `list-format.md`, `CLAUDE.md`, the named sections of
  `USER_WORKFLOW.md`, the item's entry), not those files whole. Their rules apply
  throughout; the steps below refer to the sections of `work-item.md` by letter.

**Only `## Concrete features` in `CLAUDE_PLAN.md`.** Never work on an idea from
`USER_IDEAS.md` or `CLAUDE_IDEAS.md`, even when named: point the user to
`/plan-features <id>`, which turns it into concrete features.

Arguments (all optional, see also `work-item.md` A): an item id or title (`F2`), or a focus
(`UI`, `quick`); `unattended`; `skip=<ids>`.

## 1. Check for open points

With an id, `workflow-lists show <id>` is all you read of `CLAUDE_PLAN.md`. Without one, run
`workflow-lists summary CLAUDE_PLAN.md` and `workflow-lists section CLAUDE_PLAN.md
"Suggested order"`, and read only the entries you weigh (`show`). If `## Concrete features`
holds no items, say there is nothing open (suggest `/plan-features` to turn ideas into
features); unattended, finish with `RESULT: NOTHING_LEFT`. The same result when a given id's
entry no longer exists (`show` prints `not found`: already built by another item).

## 2. Pick the item

Without an id, choose in this order of preference:

1. The suggested-order section in `CLAUDE_PLAN.md`, if there is one.
2. Items other items build on ("needs F2").
3. Higher severity ([H] > [M] > [L]), then the smaller, lower-risk change.
4. Skip items in `skip=` and items that need the user (below) without a recorded decision.
5. Never work on an item tagged **[HOLD]**, not even when named: tell the user it is on hold
   and that they can remove the tag.

Entries the user added or edited by hand can be short. Work out what they mean from the
entry, the design docs it builds on and the code. **Needs the user** (`work-item.md` B),
besides the general cases: the item contradicts the design docs and its **Decisions:** don't
say to go ahead. Attended, state which item you picked and why in one or two sentences, then
proceed (or ask, if it needs the user).

## 3. Implement, review, verify, update, commit

Follow `work-item.md` B to G:

- If the feature adds state the verification should cover (a diagnostic dump, a test),
  extend the verification.
- Update `CLAUDE_PLAN.md`.
- Commit type usually `feat`; footer `Refs: plan <id> (<title>)`.

## 4. Report

As in `work-item.md` H.
