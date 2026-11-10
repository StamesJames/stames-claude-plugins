---
name: plan-features
description: Turn the user's feature ideas from USER_IDEAS.md (all open ones, given ids, a free-text wish, or an approved claude:I idea) into concrete, buildable features in CLAUDE_PLAN.md - explore the code, ask the user the open questions up front, write the features with their decisions, delete the planned ideas and commit. Also sharpens an existing CLAUDE_PLAN.md feature (F3). Use when the user asks to plan their ideas, plan or spec out a feature, turn ideas into features, or prepare the plan for /implement-all-features. Never changes project code. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Plan features from ideas

The user writes ideas in `USER_IDEAS.md`, as short and vague as they like. This skill turns
them into entries in `CLAUDE_PLAN.md` so clear that `/implement-all-features` can build them
without asking anything: every behaviour decided, a **Done when:** the user agreed to, and a
size an agent can do in one go. Writing an entry into `CLAUDE_PLAN.md` is the user's approval
to build it, so every entry goes past the user before it is written. This skill only explores
and writes list entries; it never changes project code.

Argument (optional):

- nothing: every open idea in `USER_IDEAS.md` that is not tagged **[HOLD]**;
- ids from `USER_IDEAS.md` (`I3 I5`), or from `CLAUDE_IDEAS.md` (`claude:I2`; naming one
  counts as approving it);
- a free-text wish (`a way to undo the last move`); one handed on by `/brainstorm-feature`
  comes with **Agreed:** points that count as answered (don't ask them again);
- an id from `CLAUDE_PLAN.md` (`F3`) to sharpen that feature.

With one idea, wish or feature, go **in depth** (up to three question rounds). With several,
plan them as a **batch** (one round of questions for all, a second only where answers opened
new ones).

## 1. Context

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. `CLAUDE.md` is already in your context (read the file only if it isn't). Read
   `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`, `USER_WORKFLOW.md` (design
   docs, code standards, **Ask before**, `## Runs` for the default `features` model),
   `USER_IDEAS.md` and `CLAUDE_PLAN.md` (features the new ones may build on or duplicate). Of
   `CLAUDE_IDEAS.md`, read the named ideas and the `## Dropped` section.
3. No ideas to plan (none open, or all on hold): say so, suggest writing some into
   `USER_IDEAS.md` or `/pitch-feature` for one of Claude's, and stop.

## 2. Understand each idea

For every idea: what does the user want, as the user of the product would see it? Read the
parts of the design docs and the code it would touch: what exists, where it would plug in,
what constrains it. Stay read-only. Then work out:

- **Already there or covered:** the code already does it, or a `CLAUDE_PLAN.md` feature
  covers it. Say so in step 3; the idea is deleted in step 5 (a partly covered one is
  narrowed to the rest).
- **The features it becomes:** one or more `F` entries, each buildable by one agent in one go
  (S or M; L only if it can't be split). Split bigger ideas into features that build on each
  other ("needs F7"), each working on its own.
- **The open questions:** what the user of the product sees and does, scope (what is in and
  out), behaviour in edge cases, what counts as done; plus anything under **Ask before** or
  that contradicts the design docs. Only what changes the build: decide small details (exact
  numbers, colours, wording, layout) yourself and list them.
- **Keep open for:** other open ideas (`USER_IDEAS.md`, `CLAUDE_IDEAS.md`) the implementation
  should not block.

**Many ideas:** with more than four, keep this context small: spawn one read-only `Plan`
subagent per idea (in parallel; model `triage` from `## Runs`, omitted for `inherit`), each
told to read the plan format (`workflow-lists section
${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md "The plan"`), `CLAUDE.md`,
`USER_WORKFLOW.md`, its idea and the code it touches, and to answer only with the points
above: restatement, already-there verdict, proposed features (title, What, Done when, How,
Size), questions with options and a recommendation, Keep open for candidates. Write the entries yourself from their answers.

## 3. Ask the user

Lead the conversation; the user should mostly pick options, not write specs.

1. Show each idea in two or three plain lines: how you understand it, the features you
   propose (titles and sizes), and what you found in the code that matters (what already
   exists, what is hard). For an idea that is already there, say so.
2. Ask the open questions with `AskUserQuestion`, up to four per call, recommended option
   first, each with a short description of its consequence. Use previews for UI or
   data-format alternatives. Add `Not yet` as the last option of each idea's first question
   (keep the idea, don't plan it now). If an idea contradicts the design docs, say so plainly;
   the user decides whether to go ahead (then the entry's **Decisions:** says so) or adjust.
3. Further rounds only if answers opened new questions (in depth: three rounds at most;
   batch: two). An idea still open after that stays an idea (step 5).
4. Show the final entries in short form (id, title, the idea it came from, Done when, size,
   decisions) and ask with `AskUserQuestion`: `Write them` (recommended), `Change something`
   (adjust and ask again). Anything the user wants dropped or kept as an idea, they say here.

## 4. Write `CLAUDE_PLAN.md`

Write each entry with `workflow-lists add CLAUDE_PLAN.md --sev <H|M|L> --title "<title>"
[--model <spec>] [--before <id>]` and its body on stdin, in the full format of
`list-format.md`: **Idea:** (the idea's id and a few words, or the user's wish), **What**,
**Why**, **Done when:**, **Design docs**, **How** (with "needs F<n>" where a feature builds on
another), **Size**, **Decisions:** (the user's answers, in their words where they wrote their
own, and your listed small details, with the date), **Keep open for:** if any. `--model` only
if the pick differs from the default `features` model (`list-format.md` → "Choosing a
model"). Severity from the idea, M if it has none; if the user decided to go against the
design docs, start the title with `[CONFLICT]`. Place new entries by priority (`--before`);
if the new features depend on each other or the user gave an order, add or update a
`## Suggested order` section (one step per group, as `list-format.md` describes).

Sharpening a feature (`F3`): rewrite it in place, then `workflow-lists set F3 updated`.

`workflow-lists pass CLAUDE_PLAN.md "<n> ideas planned into <m> features (<ids>), <k> kept as
ideas."`

## 5. Update the ideas

- **Planned** (fully): `workflow-lists delete <idea id> --to <new feature id>` (the first
  feature it became), which points **Keep open for:** mentions to the feature. Already there:
  `workflow-lists delete <idea id>`.
- **Partly planned:** narrow the idea to what is left, then `workflow-lists set <id> updated`.
- **Not yet / still open:** leave it, but `workflow-lists set <id> decisions "<the answers so
  far>"`, so the next run starts from them.
- **Dropped:** a user idea only when the user says so (`workflow-lists delete <id>`). A Claude
  idea: `workflow-lists delete claude:<id> --dropped "<the user's reason>"` (ask for the
  reason in one short question if none was given).
- **Free-text wish kept as an idea:** `workflow-lists add USER_IDEAS.md --title "<title>"`
  with the body on stdin: the wish, the answers so far as **Decisions:** and the open
  questions.

Leave every other idea exactly as the user wrote it.

## 6. Commit

Fix every `error` the commands printed in what you changed (`workflow-lists check
CLAUDE_PLAN.md USER_IDEAS.md CLAUDE_IDEAS.md` after hand edits). Stage only the list files you edited, by explicit path (if one
had uncommitted edits before, say they are included). Message per `USER_WORKFLOW.md` →
`## Commits`, type `chore`, e.g. `chore: plan undo of last move (F7, F8)`; no `Refs:` footer.
Scope per "Scope reporting" in `list-format.md`. End with the attribution lines from the
system reminder.

## 7. Report

The features written (ids, titles, sizes), the ideas deleted as already done, the ideas kept
and what is still open about them, the commit, and the next step:
`/implement-next-feature <id>` for one, or `/implement-all-features` for the whole plan.
