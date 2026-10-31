---
name: explore-feature
description: Work out a feature together with the user: start from a vague wish, an idea (USER_PLAN.md I-item) or an item of Claude's plan (claude:F3, claude:I2), explore the code, ask the user the open questions, and write one or more concrete, buildable features with their decisions into USER_PLAN.md, ready for an unattended run. Use when the user wants to explore, refine, spec out or plan a feature, turn an idea into a feature, or approve items from CLAUDE_PLAN.md for implementation. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Explore a feature with the user

The goal is a feature entry so clear that `/implement-all-features` can build it without
asking anything: every behaviour decided, a **Done when:** the user agreed to, and a size an
agent can do in one go. This skill only explores and writes entries; it never changes project
code.

Argument: a free-text wish (`a way to undo the last move`), an id from `USER_PLAN.md` (`I2`, or
an `F` item to sharpen), or ids from `CLAUDE_PLAN.md` (`claude:F3`, `claude:I1 claude:F4`).
Without one, ask the user what they want to explore, offering the most promising open ideas
from both plans as options.

## 1. Context

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`, `CLAUDE.md`,
   `USER_WORKFLOW.md` (design docs, code standards, `## Runs` for the default `features`
   model), `USER_PLAN.md` and `CLAUDE_PLAN.md` (to avoid duplicates; the **Dropped** section
   lists what the user turned down).
3. Run `workflow-lists stamp` for the stamps.
4. Read the parts of the design docs and the code the feature would touch: what exists, where
   it would plug in, what constrains it. Stay read-only.

## 2. Explore with the user

Lead the conversation; the user should mostly pick options, not write specs.

1. Restate the feature in two or three plain sentences as you understand it, with what you
   found in the code that matters (what already exists, what is hard).
2. Ask the open questions with `AskUserQuestion`, up to four per call, recommended option
   first, each with a short description of its consequence. Cover what the user of the
   product sees and does, scope (what is in and out), behaviour in edge cases, and what counts
   as done. Use previews for UI or data-format alternatives. Ask only what changes the build;
   decide small details yourself and list them.
3. A second or third round only if answers opened new questions. Three rounds at most; if
   still open after that, record the rest as open questions and keep it as an idea.
4. **Size:** an entry should be buildable by one agent in one go (S or M; L only if it can't
   be split). Propose splitting bigger features into several `F` entries that build on each
   other ("needs F7"), each working on its own.
5. If the feature contradicts the design docs, say so plainly; the user decides whether to go
   ahead (then the entry's **Decisions:** says so) or adjust.
6. Check `## Ideas` in both plans for ideas the implementation should not block, and propose
   them for **Keep open for:**.

## 3. Confirm and write

Show the user the final entries in short form (id, title, Done when, decisions, size) and ask
to confirm with `AskUserQuestion`: `Write it` (recommended), `Change something` (then adjust
and ask again), `Keep it as an idea` (write or update an `I` entry with the answers so far and
the open questions), `Drop it`.

On `Write it`, in `USER_PLAN.md` → `## Concrete features`, with ids from its `_Next id:_`
line, in the full plan format of `list-format.md`: **What**, **Why**, **Done when:**, **Design
docs**, **How**, **Size**, **Decisions:** (the user's answers and your listed small details),
**Keep open for:** if any, **Model:** if it differs from the default (`list-format.md` →
"Choosing a model"). Then remove the source:

- a `USER_PLAN.md` idea: delete it (or narrow it to what is left, with `_Updated:_`);
- a `USER_PLAN.md` feature being sharpened: rewrite it in place with `_Updated:_`;
- a `CLAUDE_PLAN.md` item: delete it there with its mentions, and update references to it in
  **Keep open for:** lines of both plans.

On `Drop it` for a `CLAUDE_PLAN.md` item: delete it and add a line to its `## Dropped`
section (`- <id> <name> — <the user's reason> (<date>)`; ask for the reason in one short
question if none was given). For a user idea, delete it only if the user says so.

## 4. Commit

Run `workflow-lists check USER_PLAN.md CLAUDE_PLAN.md` and fix every `error` it reports in
what you changed. Stage only the plan files you edited, by explicit path (if one had uncommitted edits before,
say they are included). Message per `USER_WORKFLOW.md` → `## Commits`, type `chore`, e.g.
`chore: plan undo of last move (F7, F8)`; no `Refs:` footer. Scope per "Scope reporting" in
`list-format.md`. End with the attribution lines from the system reminder.

## 5. Report

The entries written (ids and titles), anything kept as an idea, and the next step:
`/implement-next-feature <id>` for one, or `/implement-all-features` for the whole plan.
