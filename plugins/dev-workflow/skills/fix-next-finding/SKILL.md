---
name: fix-next-finding
description: Fix one finding from CLAUDE_ANALYSIS.md / USER_ANALYSIS.md (the best next one, or a given id), verify it, delete its entry and commit. With `user-only`, only USER_ANALYSIS.md. Use when the user asks to fix the next finding, bug or cleanup item, or names a finding to fix. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Fix the next analysis finding

Two running lists of open cleanup findings live at the repo root: `CLAUDE_ANALYSIS.md`
(written by `/analyse-code`) and `USER_ANALYSIS.md` (written by hand by the user; its ids carry
a `U` prefix outside the file, a bare id means `CLAUDE_ANALYSIS.md`). This skill fixes **one
finding**: pick it, fix it, verify it, delete its entry and commit.

Before anything else:

- If `USER_WORKFLOW.md` is missing, the project isn't set up for this workflow: tell the user
  to run `/setup-workflow` and stop.
- Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`,
  `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/work-item.md`, `CLAUDE.md` and
  `USER_WORKFLOW.md`. Their rules apply throughout; the steps below refer to the sections of
  `work-item.md` by letter.

Arguments (all optional, see also `work-item.md` A):

- a finding id (`1.4`, `U1.1`): fix that one. A focus (`performance`, `quick`): pick within
  it. Neither: pick the best item overall.
- `user-only`: only `USER_ANALYSIS.md` exists for this run. Don't pick, fix or count anything
  from `CLAUDE_ANALYSIS.md`, and don't edit it (apart from deleting an entry that the user
  item's fix also resolved). "Nothing open" refers to the user's list alone, and the
  suggestion is to add findings rather than `/analyse-code`. A closing line saying how many
  items `CLAUDE_ANALYSIS.md` has open is fine.
- `unattended`, `skip=<ids>`.

## 1. Check for open points

Read the list(s) in play (with an id: only that entry). If sections 1–3 hold no findings (or
the files are missing), say there is nothing open (suggest `/analyse-code`); unattended,
finish with `RESULT: NOTHING_LEFT`.

## 2. Pick the finding

Without an id, choose in this order of preference:

1. **The user's findings come first.** A `USER_ANALYSIS.md` item goes before every
   `CLAUDE_ANALYSIS.md` item of the same or lower severity. Only a `CLAUDE_ANALYSIS.md` item of
   higher severity (e.g. [H] over a user [M]) goes first.
2. Within each file, **section 4 ("Suggested order of work")** is the main guide: take the
   first open entry there. Fix a bundle in one go only if the items really share the same
   change. If a file's section 4 is empty, go by severity.
3. Items that unblock others (cross-references like "see 3.2", "needed for 1.4").
4. Higher severity, then the smaller, lower-risk change.
5. Skip items in `skip=` and items that need the user (below) without a recorded decision.
6. Never work on an item tagged **[HOLD]**, not even when named: tell the user it is on hold
   and that they can remove the tag.

A short user entry (a symptom, no **Fix:**, no paths) is not a reason to skip it: find the
cause in the code yourself (reproduce it where the verification tools allow) and fix that.
If the same problem is in both lists, fix it once and delete both entries.

**Needs the user** (`work-item.md` B), besides the general cases: items marked "verify" or
"possible", and items asking about design intent. Attended, state which item you picked and
why in one or two sentences, then proceed (or ask, if it needs the user).

## 3. Fix, review, verify, update, commit

Follow `work-item.md` B to G:

- Follow the **Fix:** suggestion unless the code shows a better way; if you deviate, say why
  in the report.
- Update the analysis file the finding came from.
- Commit type follows what the change does: a bug is `fix`, a cleanup `refactor`, a
  performance finding `perf` (say what got better and by how much), a stale doc `docs`.
  Footer `Refs: analysis <id> (<title>)` or `Refs: user analysis <id> (<title>)`.

## 4. Report

As in `work-item.md` H.
