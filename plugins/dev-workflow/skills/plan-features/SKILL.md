---
name: plan-features
description: Look for ideas and concrete features that would improve the project, aligned with the design docs named in USER_WORKFLOW.md, and record them with date/time stamps in CLAUDE_PLAN.md at the repo root. Ideas that conflict with the design docs are only added when exceptionally good (and allowed), clearly marked with the conflict and the reason. Use when the user asks for feature ideas, a brainstorm, a planning pass or suggestions for what to build next. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Feature planning pass

`CLAUDE_PLAN.md` (repo root) is Claude's **running list** of ideas and concrete features
that would make the project better. Every pass updates it. This skill only thinks and records:
do not change project code, and don't commit (the user commits planning files themselves).
Items in this list are never implemented in bulk. They are pitched to the user one at a time
with `/pitch-feature`.

`USER_PLAN.md` is the user's own list with the same format. Read it but **never edit it**.

Optional argument: a focus area (e.g. a subsystem, `onboarding`, `UI`, one of the idea
sources). Without one, look at the whole project.

## 1. Gather context

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`.
3. Run `date '+%Y-%m-%d %H:%M %Z'` and `git rev-parse --short HEAD`. Every stamp written in this
   pass uses exactly these values.
4. Read `USER_WORKFLOW.md`: the **Design docs** (and whether conflicting ideas are allowed)
   and the **Idea sources**. Then read **all the design docs** it names. They are the intent,
   and the yardstick for every idea. Note what they plan that does not exist yet. If the
   project has no design docs, the intent comes from `CLAUDE.md`, the README and
   `USER_PLAN.md`; say so in the report.
5. Read `CLAUDE.md`, the current `CLAUDE_PLAN.md` and `USER_PLAN.md`, so you don't duplicate
   anything already listed by either side. The **Dropped** section of `CLAUDE_PLAN.md` lists
   ideas the user turned down: don't suggest them again, not even reworded. Only bring one back
   if the project changed so much that the user's reason no longer applies, and then say so in
   the item and in the report.
6. Look at the project as it is now: its main systems, data and UI. Ideas have to fit the code
   that exists and say how they would plug into it. If a tool to run and look at the product
   is available (see `CLAUDE.md` → `### Verification`), using it can help judge the feel; it's
   optional.

## 2. Look for ideas

Sources, in this order:

1. **Gaps against the design docs:** things they describe that the project doesn't have yet.
   Turning these into concrete, buildable features is the most valuable thing this pass does.
2. **The idea sources in `USER_WORKFLOW.md`**, in their order. If it lists none, use:
   usability, feedback and clarity of what is going on, quality of life, robustness.

Each idea must be **aligned with the design docs**: it supports, refines or fills in what they
describe, and doesn't contradict them.

**Conflicting ideas** are the exception, and only if `USER_WORKFLOW.md` allows them. Only add
one if you think it is *exceptionally* good, good enough that it would be worth changing the
plan for. At most one or two per pass, and usually none. It must be marked `[CONFLICT]` and
say exactly which part of the design docs it conflicts with (file and quote or section) and
why it is worth adding anyway.

Prefer fewer, well-thought-out items over a long list. Check each against the code (is it
already there, partly there, or impossible with the current architecture?) before recording.

## 3. Update `CLAUDE_PLAN.md`

Keep its structure and follow `list-format.md` (entry formats, ids, stamps).

- **Existing items:** keep the text and the `_Written:_` stamp. If the code shows an item is
  done, delete it along with its mentions in other items. If an item needs changing (the code
  moved on, a better approach, new info), rewrite it and add an `_Updated:_` stamp. An idea
  that has become concrete moves to `Concrete features` with a new `F` id. Leave items tagged
  **[HOLD]** as they are, apart from deleting them when they're done.
- **New items:** `F<n>` in `Concrete features` (you could write the task list now), `I<n>` in
  `Ideas` (still need design work), ids from each section's `_Next id:_` line, counted up.
- `## Dropped` is only written by `/pitch-feature`.
- Add a line to the **Passes** list in the header:
  `- <date time TZ> at <commit>: <scope>, <n> new, <m> updated, <k> removed.`

## 4. Report back

In a few short paragraphs: how many items are new, updated and removed, the best new ones in
plain words, any `[CONFLICT]` item and why you added it, and which item you'd pitch first
with `/pitch-feature`. Do not paste the whole file. Remind the user the list is uncommitted.
