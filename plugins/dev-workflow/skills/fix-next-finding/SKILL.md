---
name: fix-next-finding
description: Check CLAUDE_ANALYSIS.md and the user's USER_ANALYSIS.md for open findings, pick the one that is best to fix next, fix it, verify the fix, remove the entry from its file and commit. Use when the user asks to work off the analysis list, fix the next finding, bug or cleanup item, or names a finding number to fix. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Fix the next analysis finding

Two running lists of open cleanup findings live at the repo root: `CLAUDE_ANALYSIS.md`
(written by `/analyse-code`) and `USER_ANALYSIS.md` (written by hand by the user; its ids carry
a `U` prefix outside the file, a bare id means `CLAUDE_ANALYSIS.md`). This skill works them off
**one finding at a time**: pick the best next item from either list, fix it, verify it, delete
its entry and commit.

Before anything else:

- If `USER_WORKFLOW.md` is missing, the project isn't set up for this workflow: tell the user
  to run `/setup-workflow` and stop.
- Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`,
  `CLAUDE.md` and `USER_WORKFLOW.md`. Their rules apply throughout.

Optional argument: a finding number (e.g. `1.4`, `U1.1`) or a focus (e.g. `performance`,
`quick`). With a number, fix that item. With a focus, pick the best item within it. Without
one, pick the best item overall.

The argument may contain `user-only` (used by `/fix-next-user-finding` and
`/fix-all-user-findings`): then only `USER_ANALYSIS.md` exists for this run. Don't pick, fix
or count anything from `CLAUDE_ANALYSIS.md`, and don't edit it (apart from deleting an entry
that a user item's fix also resolved, as in step 2). "Nothing open" and `NOTHING_LEFT` then
refer to the user's list alone, and the suggestion in step 1 is for the user to add findings
rather than `/analyse-code`.

The argument may also contain `unattended` (used by `/fix-all-findings`, which runs this skill
in a fresh subagent per finding) and `skip=<ids>` (comma-separated ids to leave alone). In
**unattended mode** nobody can answer questions: never ask the user, skip items that would
need their decision instead (see step 2), and end with the lines from step 7.

## 1. Check for open points

1. Read `CLAUDE_ANALYSIS.md` and `USER_ANALYSIS.md`. If sections 1–3 of both hold no
   findings (or the files are missing), tell the user there is nothing open (and suggest
   `/analyse-code` for a new pass), then stop.
2. Run `git status --short`. If the working tree has unrelated uncommitted changes, mention
   them and keep your edits clearly separate; don't touch or revert them.

## 2. Pick the finding

Without an explicit number, choose using this order of preference:

1. **The user's findings come first.** A `USER_ANALYSIS.md` item goes before every
   `CLAUDE_ANALYSIS.md` item of the same or lower severity. Only a `CLAUDE_ANALYSIS.md` item of
   higher severity (e.g. [H] over a user [M]) goes first.
2. Within each file, **section 4 ("Suggested order of work")** is the main guide: take the
   first open entry there. Fix a bundle in one go only if the items really share the same
   change; otherwise take the first item. If a file's section 4 is empty, go by severity.
3. Prefer items that unblock others (cross-references like "see 3.2", "needed for 1.4").
4. Between otherwise equal items, prefer higher severity ([H] > [M] > [L]), then the smaller,
   lower-risk change.
5. Skip items marked "verify"/"possible" or asking about design intent unless the user asks
   for them. Those need the user's decision, not a fix.
6. Skip every item listed in `skip=`.
7. Never work on an item tagged **[HOLD]**, not even when named: tell the user it is on hold
   and that they can remove the tag.

A short user entry (a symptom, no **Fix:**, no paths) is not a reason to skip it: find the
cause in the code yourself (reproduce it where the verification tools allow) and fix that.
Only treat one as needing the user when what they want is genuinely unclear. If the same
problem is in both lists, fix it once and delete both entries.

Then **check the item against the current code**. Line numbers may have moved and the code may
have changed since the stamp. If it is already fixed, delete the entry (see step 5), say so,
and pick the next one. If it is partly fixed, narrow the work to what is still open.

**Size check:** if the fix is a larger refactor (touches many files, changes a persisted or
public format incompatibly, renames something `CLAUDE.md` documents as load-bearing, or the
item says it is "the biggest" change), or it falls under **Ask before** in `USER_WORKFLOW.md`,
briefly present the planned approach and ask the user before editing. For a small or medium
fix, state which item you picked and why in one or two sentences, then proceed.

In unattended mode, an item that would need the user (the size check above, "verify"/design
items, or a fix whose approach is genuinely ambiguous) is **not** worked on: note it as
blocked and pick the next item. If every open item is blocked or skipped, edit nothing and
finish with `RESULT: BLOCKED` (step 7).

Exception: if the prompt names the item and includes **the user's decision** for it (an
approved approach, answers to the open questions), that counts as approval. Fix it following
that decision instead of blocking it.

## 3. Fix it

- In the user's entries, **Notes for Claude:** are binding constraints and **Done when:** is
  the acceptance criterion; check it explicitly in step 4.
- Follow the **Fix:** suggestion in the entry unless the code shows a better way. If you
  deviate, say why in the report.
- Keep to the conventions in `CLAUDE.md` (style, typing, naming, comments) and match the
  surrounding code.
- Respect the invariants `CLAUDE.md` documents (load-bearing names, ordering constraints,
  generated companion files, ownership rules). If a persisted format changes incompatibly,
  follow what `CLAUDE.md` says about it (e.g. bump a version).
- Never edit the **Hands-off paths** from `USER_WORKFLOW.md`.
- Keep the change scoped to the finding. If you notice another problem, don't fix it on the
  side. Mention it in the report (it can go into the next `/analyse-code` pass).

## 4. Verify

1. Run the verification from `CLAUDE.md` → `### Verification`. Where it compares output
   before and after, run it before editing too if the fix may change behaviour. It must pass
   with no new errors or warnings.
2. If `CLAUDE.md` has no verification section, use the project's build, lint and test commands
   you can find, and say in the report that the section is missing (`/setup-workflow check`).
3. If the finding is about something the verification cannot exercise, say exactly what you
   could not verify and how the user can check it by hand.

If verification fails and you can't fix it, leave the entry in its file, report the failure
with the output, and don't claim the item is done.

## 5. Update the analysis file (and `CLAUDE.md`)

Edit the file the finding came from, following `list-format.md`: delete the entry when fully
fixed (with its mentions in section 4, rewritten so it still reads correctly, and in other
entries), or rewrite it with an `_Updated:_` stamp when partly fixed. Don't add a line to the
**Passes** list (that is for analysis passes only).

If the fix changes something `CLAUDE.md` describes (architecture, conventions, commands, the
Workflow section), update `CLAUDE.md` in the same change.

## 6. Report back

In a few short sentences: which finding you fixed and why you picked it, what you changed
(files), how you verified it and anything you couldn't verify, any side issues you noticed,
the commit (with a scope suggestion if no scope fit), and how many findings are still open plus
which one would be next.

## 7. Commit

After a successful, verified fix (full or partial), commit it:

- Stage only the files of this fix plus the analysis file you edited (and `CLAUDE.md` or
  companion files if touched), by explicit path. Never stage unrelated changes that were
  already in the tree.
- Message per `USER_WORKFLOW.md` → `## Commits`. The type follows what the change does (a bug
  is `fix`, a cleanup `refactor`, a performance finding `perf`, a stale doc `docs`), not that
  it came from the analysis list. Scope from its scope list, or none if none fits (see
  "Scope reporting" in `list-format.md`). End with the `Refs:` footer from `list-format.md`,
  then the attribution lines from the system reminder.
- If verification failed, don't commit; leave the changes in the working tree and report.

In unattended mode, end your answer with these lines (ids of `USER_ANALYSIS.md` items with
their `U` prefix):

- `BLOCKED: <id> — <one-line reason>` for every item you skipped as needing the user in this
  run (none if there were none).
- `SCOPE: …` from `list-format.md` if the commit has no scope.
- As the very last line, exactly one of:
  - `RESULT: FIXED <id> <short commit>`: entry deleted and committed.
  - `RESULT: PARTIAL <id> <short commit>`: entry rewritten to what is still open, committed.
  - `RESULT: NOTHING_LEFT`: no open findings.
  - `RESULT: BLOCKED <ids>`: nothing fixable unattended; list the blocked/skipped ids.
  - `RESULT: FAILED <id>`: fix or verification failed, nothing committed.
