---
name: analyse-code
description: Run a cleanup analysis pass over the project, looking for bugs, unclean implementations and performance problems, and record the findings with date/time stamps in CLAUDE_ANALYSIS.md at the repo root, then commit that file. Use when the user asks for a code analysis, cleanup round, audit or review of the whole project (not of a single diff). Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Code analysis pass

The goal is to keep the project in a clean state so new features are easy to add.
`CLAUDE_ANALYSIS.md` (repo root) is a **running list** of findings that gets updated by every
pass. This skill only analyses and records: do not change project code unless the user asks.
At the end it commits `CLAUDE_ANALYSIS.md` (and nothing else).

Optional argument: a focus area (e.g. a subsystem, `save system`, `performance`). Without one,
analyse the whole project.

## 1. Gather context

1. If `USER_WORKFLOW.md` is missing, the project isn't set up for this workflow: tell the user
   to run `/setup-workflow` and stop.
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`: the list
   format and rules.
3. Run `date '+%Y-%m-%d %H:%M %Z'` and `git rev-parse --short HEAD`. Every stamp written in this
   pass uses exactly these values.
4. Read `CLAUDE.md` (its `## Workflow` section gives the code scope and verification),
   `USER_WORKFLOW.md` (analysis focus, things not to flag, hands-off paths), the current
   `CLAUDE_ANALYSIS.md` and the user's `USER_ANALYSIS.md`. Never edit `USER_ANALYSIS.md` in a
   pass and don't copy its items into `CLAUDE_ANALYSIS.md`. If your analysis finds the cause of
   one of the user's items, or shows one is already fixed, say so in the report instead.
5. If there was an earlier pass (the **Passes** list), run
   `git log --oneline <last pass commit>..HEAD` and `git diff --stat <last pass commit>..HEAD`
   to see what changed since then.

## 2. Analyse

Read the code itself, not only excerpts: everything in the code scope from `CLAUDE.md`, plus
the config and data files a finding depends on. Skip generated code and the hands-off paths
from `USER_WORKFLOW.md`. Look for:

- **Bugs:** wrong behaviour, edge cases, persistence and load gaps, ordering of events or
  initialisation, failures only in release/export/production builds, error handling.
- **Performance:** work per frame/request/item that scales with data size, needless
  allocations, blocking I/O on hot paths, event storms.
- **Cleanliness/architecture:** duplication, misplaced responsibilities, fragile coupling
  (load-bearing names, implicit ordering), naming clashes, dead code or assets, stale comments,
  and a stale `CLAUDE.md`.
- Everything listed under **Analysis focus** in `USER_WORKFLOW.md`, weighted as it says.

Verify before recording. Check claims against the code and, where it helps, run the
verification from `CLAUDE.md` → `### Verification`. Mark anything you could not confirm as
"verify" or "possible" rather than stating it as fact. Do not record what
`USER_WORKFLOW.md` → **Don't flag** lists, nor conventions `CLAUDE.md` documents on purpose.

## 3. Update `CLAUDE_ANALYSIS.md`

Keep its structure and follow `list-format.md`.

- **Existing findings:** leave the text and the `_Found:_` stamp as they are. If the code shows
  one is resolved, delete the whole entry with its mentions. If one is only partly resolved, or
  changed, rewrite it to what is still open and add an `_Updated:_` line. Update line numbers
  that moved.
- **New findings:** append each to its section with the id from that section's `_Next id:_`
  line, counting it up, in the entry format from `list-format.md`.
- Add a line to the **Passes** list in the header: `- <date time TZ> at <commit>: <scope>, <n>
  new, <m> fixed.`
- Rewrite section 4 (suggested order) so it reflects the open items.

## 4. Commit

Commit the updated list:

- Stage only `CLAUDE_ANALYSIS.md`, by explicit path. Never stage other changes that are in the
  tree (if `CLAUDE_ANALYSIS.md` itself already had uncommitted edits before this pass, mention
  that they are included).
- Message per `USER_WORKFLOW.md` → `## Commits`: type `chore` (the workflow list files), scope
  from its scope list or none if none fits (see "Scope reporting" in `list-format.md`), e.g.
  `chore: record analysis pass (<n> new, <m> fixed)`. No `Refs:` footer (the pass doesn't
  resolve items by changing code). End with the attribution lines from the system reminder.

## 5. Report back

Tell the user, in a few short paragraphs: how many findings are new, how many are fixed and how
many are still open, the most important new ones in plain words, the commit (with a scope
suggestion if no scope fit), and the recommended next step. Do not paste the whole file.
