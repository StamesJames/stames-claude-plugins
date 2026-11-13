---
name: review-change
description: Review a range of commits (by default the last commit) with a fresh eye - first whether it does what its list item asked (Done when, Verify, the verification), then clean code against the project's code standards (reuse, simplicity, naming, structure, tests) - apply the clear improvements without changing behaviour, verify and commit them as a follow-up, or send the item back for rework. Used by the workflow runs after items; use it directly when the user asks to review, check or tidy up a recent commit or change.
---

# Review a change

You review code someone else just wrote, with a fresh context: the point is a second look
that isn't anchored on how the change came about. First you check that it **does what was
asked** and that the verification really passes; then you improve **the changed code only**,
without changing what it does. In a run you are the acceptance gate: a change that doesn't
meet its entry goes back for rework instead of being tidied.

Arguments (all optional): a commit range (`<from>..<to>`, default `HEAD~1..HEAD`), the list
item it implemented (`F2`, `1.4`, `U1.1`), `unattended` (used by the runs: never ask, keep
the answer short, end with the lines from step 5), and `final` (the last rework round is
used up: you can't send it back; fix what you can and report the rest).

## 1. Context

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. `git status --short` must be clean; otherwise say so and stop (unattended:
   `RESULT: FAILED dirty tree`).
3. `CLAUDE.md` is already in your context (read the file only if it isn't); use it for
   conventions, invariants and `### Verification`. `workflow-lists brief --context` prints the
   item rules, the `USER_WORKFLOW.md` sections that apply and the verification; read nothing
   else of those files.
4. `git log --format='%h %s' <range>` and `git diff <range>`. The item's entry was deleted in
   the range: its text is in the diff of the list file (its **Done when:**, **Verify:**,
   **Decisions:**, **Notes for Claude:** and **Keep open for:** still apply). Without an item,
   the commit messages say what was intended.
5. Read the touched files around the changes, and search the project for existing helpers or
   patterns the new code could have reused.

## 2. Acceptance

1. Run the verification yourself, and the entry's **Verify:** steps; don't trust the
   implementer's report. If `### Verification` has tiers, run the set the item agent ran once
   (its `CHECKS:` line, appended to the prompt in a run; without one, choose by the tier rule
   in the item rules), plus whatever the diff needs beyond it. A set that was clearly too
   light for the diff (a touched area's Targeted check, or Full in its named cases, left out)
   is a gap; with `final`, run what was missing and note it.
2. Check **Done when:** against the diff and the running code where you can: is every part
   of it there, do **Decisions:** and **Notes for Claude:** hold, does anything in the change
   do what **Keep open for:** says not to block?
3. Bugs in the changed code: wrong edge cases, broken callers, behaviour changed where the
   item didn't intend it.

A gap here is **not met**. Unattended and not `final`: don't fix it yourself (even if small),
list the gaps and go to step 5 with `REWORK`. Attended: say what is missing and ask whether
to fix it here or send it back. With `final`: fix what is small and clear, report the rest as
`NOTE:` lines.

## 3. Clean code

With the acceptance met, look at the changed lines against `## Code standards` and the
surrounding code:

- **Reuse:** new code that duplicates something the project already has, or repeats itself.
- **Simplicity:** needless indirection, abstractions or options nothing uses (also those
  built "for" a **Keep open for:** item), convoluted control flow, dead or debug code.
- **Fit:** names, structure, error handling and placement that differ from the conventions,
  responsibilities in the wrong module.
- **Efficiency:** obvious waste on hot paths introduced by the change.
- **Tests and docs:** missing tests for the new behaviour where the project has a test suite,
  stale comments, a `CLAUDE.md` that no longer matches.

Only **clear** improvements count. Not taste, not rewrites, and not old code outside the
change (report notable problems there as `NOTE:` lines; they belong in `/analyse-code`).
Missing tests for new behaviour are a gap for step 2 when the project has a test suite and
`## Code standards` asks for them: send it back rather than writing them yourself, unless
`final`.

## 4. Apply and commit

If nothing clearly improves the change, edit nothing and go to step 5. Otherwise make the
improvements, never touching hands-off paths, then verify again: only **Quick** and the
**Targeted** checks of the files you touched, never **Full** (untiered: all of it). If it fails
and you can't fix it quickly, undo all your edits (`git restore` / remove files you created)
and report `FAILED`.

Stage only the files you changed, by explicit path. Message per `## Commits`, type
`refactor` (or `test` if you only added tests), e.g.
`refactor(ui): simplify pause menu sound setup`. If you know the item, end with its `Refs:`
footer with `, review` inside the parentheses, then the attribution lines from the system
reminder.

## 5. Report

Attended: whether the change does what was asked, which checks you ran, what you changed and
why, in a few lines; problems you saw but didn't fix; the commit.

Unattended: at most three plain lines, then `CHECKS: <id> — <tiers and checks run> (<why>)`,
`NOTE: <id> — <problem seen, not fixed>` lines, `SCOPE: …` if the commit has no scope, and as
the very last line exactly one of:

- `RESULT: ACCEPTED [<short commit>]`: the change does what was asked; the commit if you
  tidied something.
- `RESULT: REWORK — <the gaps, one line>`: not met; nothing committed. The run gives the item
  one more round with this line.
- `RESULT: FAILED <one-line reason>`: edits undone, nothing committed.
