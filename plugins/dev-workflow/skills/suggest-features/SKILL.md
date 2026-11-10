---
name: suggest-features
description: Brainstorm Claude's own feature ideas that would improve the project, aligned with the design docs named in USER_WORKFLOW.md, and record them in CLAUDE_IDEAS.md, then commit that file. Nothing is planned or built from it without the user's approval (/pitch-feature). Use when the user asks for Claude's feature ideas, a brainstorm or suggestions for what to build next. Not for the user's own ideas: talking one through is /brainstorm-feature, planning them is /plan-features. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Feature suggestion pass

`CLAUDE_IDEAS.md` (repo root) is Claude's **running list** of ideas that would make the
project better. Every pass updates it. This skill only thinks and records: do not change
project code. At the end it commits `CLAUDE_IDEAS.md` (and nothing else). Ideas in this list
are only planned with the user's approval: `/pitch-feature` pitches one and moves it to
`USER_IDEAS.md`, where `/plan-features` turns it into concrete features.

`USER_IDEAS.md` (the user's ideas) and `CLAUDE_PLAN.md` (features already planned) are read
but **never edited** here.

Optional argument: a focus area (e.g. a subsystem, `onboarding`, `UI`, one of the idea
sources). Without one, look at the whole project.

## 1. Gather context

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`.
3. Run `workflow-lists stamp`. Every stamp written in this pass uses exactly this value
   (and the **Passes** line its date, time and commit).
4. Read `USER_WORKFLOW.md`: the **Design docs** (and whether conflicting ideas are allowed)
   and the **Idea sources**. Then read **all the design docs** it names. They are the intent,
   and the yardstick for every idea. Note what they plan that does not exist yet. If the
   project has no design docs, the intent comes from `CLAUDE.md`, the README and
   `USER_IDEAS.md`; say so in the report.
5. Use `CLAUDE.md` (already in your context; read the file only if it isn't). Read the current
   `CLAUDE_IDEAS.md`, `USER_IDEAS.md` and `CLAUDE_PLAN.md`, so you don't duplicate anything
   already listed. The **Dropped** section of `CLAUDE_IDEAS.md` lists ideas the user turned
   down: don't suggest them again, not even reworded. Only bring one back if the project
   changed so much that the user's reason no longer applies, and then say so in the idea and in
   the report.
6. Look at the project as it is now: its main systems, data and UI. Ideas have to fit the code
   that exists and say how they would plug into it. If a tool to run and look at the product
   is available (see `CLAUDE.md` → `### Verification`), using it can help judge the feel; it's
   optional.

## 2. Look for ideas

Sources, in this order:

1. **Gaps against the design docs:** things they describe that the project doesn't have yet.
   These are the most valuable suggestions this pass makes.
2. **The idea sources in `USER_WORKFLOW.md`**, in their order. If it lists none, use:
   usability, feedback and clarity of what is going on, quality of life, robustness.

Each idea must be **aligned with the design docs**: it supports, refines or fills in what they
describe, and doesn't contradict them.

**Conflicting ideas** are the exception, and only if `USER_WORKFLOW.md` allows them. Only add
one if you think it is *exceptionally* good, good enough that it would be worth changing the
plan for. At most one or two per pass, and usually none. It must be marked `[CONFLICT]` and
say exactly which part of the design docs it conflicts with (file and quote or section) and
why it is worth adding anyway.

Prefer fewer, well-thought-out ideas over a long list. Check each against the code (is it
already there, partly there, or impossible with the current architecture?) before recording.

## 3. Update `CLAUDE_IDEAS.md`

Keep its structure and follow `list-format.md` (entry format, ids, stamps).

- **Existing ideas:** keep the text and the `_Written:_` stamp. If the code shows an idea is
  done, or `CLAUDE_PLAN.md` / `USER_IDEAS.md` now cover it, delete it along with its mentions.
  If an idea needs changing (the code moved on, a better approach, new info), rewrite it and
  add an `_Updated:_` stamp. Leave ideas tagged **[HOLD]** as they are, apart from deleting
  them when they're done.
- **New ideas:** `I<n>` in `## Ideas`, ids from its `_Next id:_` line, counted up. Each with
  **What**, **Why**, **Design docs**, **Open questions** (what the user would have to decide;
  "none" if nothing), and a rough **How:** where you already see one.
- `## Dropped` is only written by `/pitch-feature` and `/plan-features`.
- Add a line to the **Passes** list in the header:
  `- <date time TZ> at <commit>: <scope>, <n> new, <m> updated, <k> removed.`
- Run `workflow-lists check CLAUDE_IDEAS.md` and fix every `error` it reports.

## 4. Commit

Commit the updated list:

- Stage only `CLAUDE_IDEAS.md`, by explicit path. Never stage other changes that are in the
  tree (if `CLAUDE_IDEAS.md` itself already had uncommitted edits before this pass, mention
  that they are included).
- Message per `USER_WORKFLOW.md` → `## Commits`: type `chore` (the workflow list files), scope
  from its scope list or none if none fits (see "Scope reporting" in `list-format.md`), e.g.
  `chore: record suggestion pass (<n> new, <m> updated, <k> removed)`. No `Refs:` footer (the
  pass doesn't implement items). End with the attribution lines from the system reminder.

## 5. Report back

In a few short paragraphs: how many ideas are new, updated and removed, the best new ones in
plain words, any `[CONFLICT]` idea and why you added it, the commit (with a scope suggestion
if no scope fit), and which idea you'd pitch first with `/pitch-feature`. Do not paste the
whole file.
