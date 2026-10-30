---
name: review-change
description: Review a range of commits (by default the last commit) with a fresh eye for clean code against the project's code standards (reuse, simplicity, naming, structure, tests), apply the clear improvements without changing behaviour, verify and commit them as a follow-up. Used by the workflow loops after each larger item; use it directly when the user asks to review or tidy up a recent commit or change. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Review and tidy a change

You review code someone else just wrote, with a fresh context: the point is a second look
that isn't anchored on how the change came about. You improve **the changed code only**,
without changing what it does.

Arguments (all optional): a commit range (`<from>..<to>`, default `HEAD~1..HEAD`), the list
item it implemented (`F2`, `1.4`, `U1.1`), and `unattended` (used by the loops: never ask,
keep the answer short, end with the lines from step 5).

## 1. Context

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. `git status --short` must be clean; otherwise say so and stop (unattended:
   `RESULT: FAILED dirty tree`).
3. Read `CLAUDE.md` (conventions, invariants, `### Verification`), `USER_WORKFLOW.md`
   (`## Code standards`, `## Hands-off paths`, `## Commits`), and the sections "Commit
   footer" and "Scope reporting" of `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`.
4. `git log --format='%h %s' <range>` and `git diff <range>`. The item's entry was deleted in
   the range: its text is in the diff of the list file (its **Done when:**, **Decisions:**
   and **Keep open for:** still apply).
5. Read the touched files around the changes, and search the project for existing helpers or
   patterns the new code could have reused.

## 2. Review

Look at the changed lines, judged against `## Code standards` and the surrounding code:

- **Reuse:** new code that duplicates something the project already has, or repeats itself.
- **Simplicity:** needless indirection, abstractions or options nothing uses (also those
  built "for" a **Keep open for:** item), convoluted control flow, dead or debug code.
- **Fit:** names, structure, error handling and placement that differ from the conventions,
  responsibilities in the wrong module.
- **Efficiency:** obvious waste on hot paths introduced by the change.
- **Tests and docs:** missing tests for the new behaviour where the project has a test suite,
  stale comments, a `CLAUDE.md` that no longer matches.

Only **clear** improvements count. Not taste, not rewrites, and not old code outside the
change (mention notable problems there in the report; they belong in `/analyse-code`). Bugs
you find are reported, not fixed here, unless the fix is tiny and obvious.

## 3. Apply

If nothing clearly improves the change, edit nothing and go to step 5. Otherwise make the
improvements, never touching hands-off paths, then run the verification from `CLAUDE.md` →
`### Verification`. If it fails and you can't fix it quickly, undo all your edits
(`git restore` / remove files you created) and report `FAILED`.

## 4. Commit

Stage only the files you changed, by explicit path. Message per `USER_WORKFLOW.md` →
`## Commits`, type `refactor` (or `test` if you only added tests), e.g.
`refactor(ui): simplify pause menu sound setup`. If you know the item, end with its `Refs:`
footer with `, review` inside the parentheses, then the attribution lines from the system
reminder.

## 5. Report

Attended: what you changed and why, in a few lines; problems you saw but didn't fix; the
commit.

Unattended: at most three plain lines, then `SCOPE: …` if the commit has no scope, and as the
very last line exactly one of:

- `RESULT: REVIEWED <short commit>`: improvements committed.
- `RESULT: CLEAN`: nothing worth changing.
- `RESULT: FAILED <one-line reason>`: edits undone, nothing committed.
