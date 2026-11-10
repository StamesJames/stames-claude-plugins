---
name: analyse-code
description: Run a cleanup analysis pass looking for bugs, unclean code and performance problems, and record the findings in CLAUDE_ANALYSIS.md, then commit that file. By default only what changed since the last pass; `full` analyses the whole project (split across one subagent per area when it is large). Use when the user asks for a code analysis, cleanup round, audit or review of the project (not of a single diff). Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Code analysis pass

The goal is to keep the project in a clean state so new features are easy to add.
`CLAUDE_ANALYSIS.md` (repo root) is a **running list** of findings that gets updated by every
pass. This skill only analyses and records: do not change project code unless the user asks.
At the end it commits `CLAUDE_ANALYSIS.md` (and nothing else).

Arguments (all optional):

- `full`: analyse the whole code scope.
- `since=<commit>`: analyse what changed since that commit (used by the loops after a run).
- a focus area (a subsystem, `save system`, `performance`): analyse only that, in full.
- `after-run` (with `since=`, from the loops): the changes were already reviewed per item;
  weight the checks as `part.md` describes for `after-run`.
- `parts=<dir>` (from the loops): the scope was already read by part agents, whose findings
  are the files in `<dir>`; merge those (step 3) instead of reading the scope yourself.
- `unattended` (run in a subagent by a loop): never ask, end with the lines from step 6.

Without `full`, `since=` or a focus, the pass is **incremental**: what changed since the
commit of the last pass in the **Passes** list. If that commit no longer exists
(`git cat-file -e <commit>` fails, e.g. after a rebase or a squash merge), use the last
commit before the day of that pass instead
(`git rev-list -1 --before='<pass date> 00:00' HEAD`). Only if there was no pass yet, or no
such commit exists, is the pass `full`; attended, say so in one line before starting (it may
be large).

## 1. Gather context

1. If `USER_WORKFLOW.md` is missing, the project isn't set up for this workflow: tell the user
   to run `/setup-workflow` and stop.
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md` (the list format and
   rules) and `${CLAUDE_PLUGIN_ROOT}/skills/analyse-code/part.md` (the checks).
3. Run `workflow-lists stamp`. Every stamp written in this pass uses exactly this value
   (and the **Passes** line its date, time and commit).
4. Use `CLAUDE.md` (already in your context; read the file only if it isn't); its `## Workflow`
   section gives the code scope and verification. Read `USER_WORKFLOW.md` (analysis focus,
   things not to flag, code standards, hands-off paths, `## Runs` for the default `fixes`
   model), the current `CLAUDE_ANALYSIS.md` and the user's `USER_ANALYSIS.md`. Never edit
   `USER_ANALYSIS.md` in a pass and don't copy its items into `CLAUDE_ANALYSIS.md`. If your
   analysis finds the cause of one of the user's items, or shows one is already fixed, say so
   in the report instead.

## 2. Decide what to read

With `parts=`, skip this step: the parts are read.

- **Incremental / `since=`:** `git diff --stat <base>..HEAD` and `git log --oneline
  <base>..HEAD`. The scope is the changed files in the code scope, plus direct users of what
  changed (callers of changed functions, readers of changed data) where the change could
  break them: at most about ten such files for the whole pass, however many commits it
  covers (pick those most likely to break); if there are more, note the rest in the report. Existing findings in those files get re-checked; the others stay
  as they are. If nothing in the code scope changed, record an empty pass.
- **Focus:** everything that belongs to the focus area.
- **`full`:** the whole code scope from `CLAUDE.md`, plus the config and data files a finding
  depends on.

Always skip generated code and the hands-off paths from `USER_WORKFLOW.md`.

**Large scopes are split.** A scope above roughly 40 files or 8000 lines is split (the usual
case for `full`); a smaller one, even a small `full`, is read directly in one context. For a
split, map the scope to the project's areas (the
`### Scopes` in `USER_WORKFLOW.md`; split big areas, merge tiny ones, 3–8 parts) and spawn one
`general-purpose` subagent per part (model `analysis` from `## Runs`; omit for `inherit`),
in parallel, with this prompt:

> Read-only analysis; don't edit any file. Read
> `${CLAUDE_PLUGIN_ROOT}/skills/analyse-code/part.md` and follow it as a part agent for
> <paths> <`since <base>` for an incremental pass> <`after-run` if given>.

When run unattended inside a subagent (which can't start agents), read the scope yourself;
the loops split large scopes themselves and pass `parts=`.

## 3. Analyse

Apply the checks of `part.md` to the scope (with the `after-run` weighting if given), and
verify before recording as it says.

With part findings (subagents, or the files of `parts=`): merge them, drop duplicates (of
each other and of open entries), check each `CROSS:` note yourself, and spot-check the cited
lines of every [H] finding. Existing findings in the files the parts covered get re-checked
as in step 2.

## 4. Update `CLAUDE_ANALYSIS.md`

Keep its structure and follow `list-format.md`.

- **Existing findings** (those in the scope): leave the text and the `_Found:_` stamp as they
  are. If the code shows one is resolved, delete the whole entry with its mentions. If one is
  only partly resolved, or changed, rewrite it to what is still open and add an `_Updated:_`
  line. Update line numbers that moved.
- **New findings:** append each to its section with the id from that section's `_Next id:_`
  line, counting it up, in the entry format from `list-format.md`, with a `**Model:**` line
  when the rubric's pick differs from the default `fixes` model.
- Add a line to the **Passes** list in the header: `- <date time TZ> at <commit>: <mode and
  scope>, <n> new, <m> fixed.` (mode: `full`, `since <commit>`, or the focus).
- Rewrite section 4 (suggested order) so it reflects the open items.
- Run `workflow-lists check CLAUDE_ANALYSIS.md` and fix every `error` it reports.

## 5. Commit

- Stage only `CLAUDE_ANALYSIS.md`, by explicit path. Never stage other changes that are in the
  tree (if `CLAUDE_ANALYSIS.md` itself already had uncommitted edits before this pass, mention
  that they are included).
- Message per `USER_WORKFLOW.md` → `## Commits`: type `chore`, scope from its scope list or
  none if none fits (see "Scope reporting" in `list-format.md`), e.g.
  `chore: record analysis pass (<n> new, <m> fixed)`. No `Refs:` footer. End with the
  attribution lines from the system reminder.

## 6. Report back

Attended, in a few short paragraphs: the mode and scope, how many findings are new, fixed and
still open, the most important new ones in plain words, the commit (with a scope suggestion if
no scope fit), and the recommended next step (usually `/fix-all-findings`). Do not paste the
whole file.

Unattended: at most three plain lines (the most important new findings), `SCOPE: …` if the
commit has no scope, and as the very last line
`RESULT: ANALYSED <n> new, <m> fixed, <k> open <short commit>`.
