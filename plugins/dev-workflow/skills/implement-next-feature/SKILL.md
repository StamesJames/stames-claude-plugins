---
name: implement-next-feature
description: Implement one concrete feature from USER_PLAN.md (the best next one, or a given id), verify it, delete its entry and commit. Never CLAUDE_PLAN.md and never Ideas. Use when the user asks to implement the next feature from their plan or names a USER_PLAN.md item to build. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Implement the next user plan item

`USER_PLAN.md` (repo root) is the list of features to build: the user's own entries and the
ones they approved from Claude's suggestions. This skill implements **one concrete feature**:
pick it, build it, verify it, delete its entry and commit.

Before anything else:

- If `USER_WORKFLOW.md` is missing, the project isn't set up for this workflow: tell the user
  to run `/setup-workflow` and stop.
- Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`,
  `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/work-item.md`, `CLAUDE.md` and
  `USER_WORKFLOW.md`. Their rules apply throughout; the steps below refer to the sections of
  `work-item.md` by letter.

**Only `## Concrete features` in `USER_PLAN.md`.** Never pick from `CLAUDE_PLAN.md` (its items
are approved into `USER_PLAN.md` with `/pitch-feature` or `/explore-feature` first). Never work
on an item from `## Ideas`, even when named: point the user to `/explore-feature <id>`, which
turns it into a concrete feature.

Arguments (all optional, see also `work-item.md` A): an item id or title (`F2`), or a focus
(`UI`, `quick`); `unattended`; `skip=<ids>`.

## 1. Check for open points

Read `USER_PLAN.md` (with an id: only that entry). If `## Concrete features` holds no items,
say there is nothing open (suggest `/explore-feature` to turn ideas into features);
unattended, finish with `RESULT: NOTHING_LEFT`.

## 2. Pick the item

Without an id, choose in this order of preference:

1. A suggested-order section in `USER_PLAN.md`, if the user wrote one.
2. Items other items build on (cross-references like "needs F2").
3. Higher severity ([H] > [M] > [L]), then the smaller, lower-risk change.
4. Skip items in `skip=` and items that need the user (below) without a recorded decision.
5. Never work on an item tagged **[HOLD]**, not even when named: tell the user it is on hold
   and that they can remove the tag.

The user's entries can be short. Work out what they mean from the entry, the design docs it
builds on and the code. **Needs the user** (`work-item.md` B), besides the general cases: the
item contradicts the design docs. Attended, state which item you picked and why in one or two
sentences, then proceed (or ask, if it needs the user).

## 3. Implement, review, verify, update, commit

Follow `work-item.md` B to G:

- If the feature adds state the verification should cover (a diagnostic dump, a test),
  extend the verification.
- Update `USER_PLAN.md`.
- Commit type usually `feat`; footer `Refs: user plan <id> (<title>)`.

## 4. Report

As in `work-item.md` H.
