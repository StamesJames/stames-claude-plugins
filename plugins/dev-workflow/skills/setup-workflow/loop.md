# Working off a whole list: shared run protocol

Read by the loop skills (`fix-all-findings`, `implement-all-features`). They name the lists,
the single-item skill, the kind of work (`fixes` / `features`), their own triage cases and
how deep triage looks at the code; everything else is here.

The user asked for this run, so spawning subagents is intended. You are the
**orchestrator**: you never read or change project code. Each item is built by a fresh
subagent, so every item starts with an empty context and only short result lines collect
here. Subagents can't start further agents, so the loop runs in this session. The only files
you edit are the list files (decisions, done items and failure stamps), always through
`workflow-lists set` / `delete`, and the run log.

**Paths in prompts.** This file is read, not loaded as a skill, so `${CLAUDE_PLUGIN_ROOT}` is
not substituted and is not set in Bash. Before sending a prompt, replace it with the plugin
root (the directory two levels above this file, which holds `skills/`).

All questions come **before** the long unattended part (section 2). After that the run doesn't
need the user, apart from the rare item that turns out to need a decision mid-run (section 6).
All state that matters is written to the list files and the run log, so an interrupted run
can simply be started again.

**Run log.** `<run dir>` is `$(git rev-parse --git-path workflow-run)` (inside `.git`, so it
never dirties the tree); the log is `<run dir>/<kind>.log`. Append one line per event with
`printf '%s\n' '<line>' >> <log>`, right when it happens: `start <commit> <branch>`,
`skip <ids>`, `gone <id> — <why>`, `unverified <id> — <what>`, the `BLOCKED:`, `DEFAULTS:`,
`SCOPE:` and `RESULT:` lines of every agent (prefixed with the id for review agents: `review <id> RESULT: …`),
`failed <id> <branch> — <reason>`, and `end` when the run is over. The final report is built
from the log, so nothing has to survive in your context; don't keep a second copy of it
there. If the log can't be written, keep the same lines in mind instead.

## 1. Before the run

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. `git status --short` must be clean. If it isn't, show the user the changes and stop: each
   item commits, and unrelated changes would get in the way.
3. Read only the `## Runs` section of `USER_WORKFLOW.md`
   (`sed -n '/^## Runs/,/^## [^R]/p' USER_WORKFLOW.md`). Missing settings use these defaults,
   and the final report suggests `/setup-workflow check`:
   - models: inherit the session's model for everything; escalation: none;
   - fresh review: features of size M or L, and any item whose diff has more than 150 changed
     lines;
   - analyse after run: features yes, fixes no;
   - stop after 3 consecutive failures;
   - run branch: no.
4. **Log.** If `<log>` exists and its last line isn't `end`, a run was interrupted: say so
   in one line, take its `start` commit and `skip` ids over, and append to it (skip the `start`
   line of step 1.6). Otherwise remove any old log and start a new one.
5. Count the open items with `workflow-lists summary <lists>` (don't read the lists for
   this): its `TOTAL:` line gives open and held items, its item lines the ids and line
   numbers. None open besides held ones: say so and stop. Run `workflow-lists check <lists>`
   too; errors in the lists go into the final report, and the run goes ahead unless they
   make items unreadable.
6. Note the branch (`git branch --show-current`) and `git rev-parse --short HEAD` as the
   start, and log `start <commit> <branch>`. If **run branch** is `yes` and the current
   branch is the default branch (`main`, `master`, or what
   `git symbolic-ref --short refs/remotes/origin/HEAD` names), create and switch to `workflow/<YYYY-MM-DD>-<kind>` (append `-2`, `-3` if it exists). The run commits
   to the current branch from here on; call it the **run branch**.
7. Keep in mind only `queue`, the escalation state of the current item and
   `consecutive_failures = 0`; log `skip <ids>` for ids from the argument. Everything else
   (done, blocked, failed, defaults, scopes, reviews) is in the log.

**Models.** Every subagent gets the model for its kind of work from `## Runs` (`triage`,
`fixes`, `features`, `review`, `analysis`); an item's `**Model:**` line overrides it. Pass it
as the Agent tool's `model` (omit it for `inherit`), and a reasoning level, if one is given,
as `effort`.

## 2. Triage and decisions

1. **Triage.** Run `workflow-lists triage <lists> --skip <skipped ids>` (in section 6 with
   `--only <ids>`). It prints `ORDER:` (the order of work: suggested order, what others build
   on, severity, size; the user's findings first), then one line per item: `READY` (workable
   as written), `FAILED` (has a `_Failed:_` stamp: a DECIDE item with the fixed question
   below), `CHECK` (needs a look at the code, with the reason), and `HELD` / `SKIPPED` ids.
   Only if there are `CHECK` items, spawn one `Plan` subagent (read-only, model `triage`),
   description `Triage <kind>`, with this prompt, adding the skill's own triage cases:

   > Read part 1 of `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md` (up to
   > `# Part 2`), sections A and B of
   > `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/work-item.md`, `CLAUDE.md`, and of
   > `USER_WORKFLOW.md` only
   > `workflow-lists section USER_WORKFLOW.md "Design docs" "Ask before" "Hands-off"`. Classify only these items, read with `workflow-lists show <ids>`:
   > <the CHECK lines>. Check each against the current code only as deeply as needed to
   > classify it (<skill's triage depth>): READY (workable unattended, including items whose
   > **Decisions:** cover their open questions), DECIDE (needs the user per section B, plus:
   > <skill's cases>), or GONE (already done or obsolete). Don't plan READY items. For each
   > DECIDE item, work out the approach and the concrete decisions the user must make. Answer
   > only in this format, one line per item:
   >
   > `READY: <id>`
   > `DECIDE: <id> | <title> — <the planned approach in one or two lines>`
   > `  Q: <question> | <recommended option> | <option> [| <option>]` (one to three per item)
   > `GONE: <id> — <why>`

   For a `FAILED` item the only question is `How to continue with <id>?` with the options
   `Retry from scratch`, `Continue from the WIP branch`, `Put on hold`, `Drop it`
   (`Continue from the WIP branch` only if it names a branch); show its reason.

2. **Ask.** If there are DECIDE items, show the user each one in two lines (id, title,
   approach), then ask their questions with `AskUserQuestion`, up to four questions per call,
   recommended option first. Add `Skip this run` as the last option of each item's first
   question; if it already has four options (a `_Failed:_` item), `Drop it` gives way, still
   reachable through "Other". Ask nothing else: no run
   confirmation, no model choice.
3. **Record.** For every DECIDE and GONE item:
   - `Skip this run`: no edit; log `skip <id>`.
   - `Put on hold`: `workflow-lists set <id> hold`; log `skip <id>`.
   - `Drop it`: `workflow-lists delete <id>`.
   - Every GONE item (no question asked): `workflow-lists delete <id>` and log
     `gone <id> — <why>`. Triage already checked the code, so no agent is spent on it; the
     final report lists them, so the user can restore one from git if triage was wrong.
   - Otherwise `workflow-lists set <id> decisions "<the answers, in the user's words where
     they wrote their own>"`. For a `_Failed:_` item it moves the stamp (branch, model,
     reason) into the decision line.

   Fix every `error` the commands print. Commit the list files you edited (by explicit path;
   type `chore`, `chore: record run decisions for <ids>` or `chore: remove done items <ids>`,
   both parts when both apply; no `Refs:` footer, scope per `list-format.md`).
4. **Queue:** `ORDER` without skipped, held, dropped and gone ids. Tell the user in one line
   how many items the run will do and that it now runs unattended.

## 3. Each iteration

1. Take the next id from `queue`. Note `before = git rev-parse --short HEAD`.
2. Spawn one `general-purpose` subagent (not `fork`) with the item's model, description
   `<Fix|Implement> <id>`, and this prompt:

   > You are working in the repo at the current directory, unattended: nobody will answer
   > questions. Invoke the `dev-workflow:<single-item skill>` skill with the argument
   > `<id> unattended <skill extras>` (if the Skill tool is unavailable, read
   > `${CLAUDE_PLUGIN_ROOT}/skills/<single-item skill>/SKILL.md` and follow it exactly). Work
   > on exactly that item, verify it, update its list file, commit, and end with the lines
   > that skill defines.

3. Wait for its completion notification. Never run two item agents at once (they would edit
   the same files). Don't poll and don't predict its result.
4. Log its `BLOCKED:`, `DEFAULTS:`, `SCOPE:` and `RESULT:` lines, and what it says it
   couldn't verify as `unverified <id> — …`. Then:
   - `DONE <id> <commit>`: reset `consecutive_failures`, run the review (section 4). Give the
     user one line (`✓ F2 Pause menu sound`) plus the defaults chosen.
   - `PARTIAL <id> <commit>`: as `DONE`; put the id at the end of `queue` once. A second
     `PARTIAL` for the same id: log `skip <id>`.
   - `BLOCKED <id>`: nothing more (the logged `BLOCKED:` line holds the reason).
   - `NOTHING_LEFT`: the item was gone; carry on.
   - `FAILED <id>`, or no `RESULT:` line: section 5 (failures; the run goes on).
5. Except after a failure, `git status --short` must be clean. If it isn't, stop and report
   what is left.
6. Stop when the maximum number of items from the argument is reached. When `queue` is empty,
   go to section 6.

## 4. Review

Decide from `## Runs` → **fresh review** whether the item gets one: its triage size, and the
changed lines of `git diff --shortstat <before>..HEAD -- . ':!*_PLAN.md' ':!*_IDEAS.md' ':!*_ANALYSIS.md'`.
If it does, spawn a `general-purpose` subagent (model `review`), description `Review <id>`:

> You are working in the repo at the current directory, unattended. Invoke the
> `dev-workflow:review-change` skill with the argument `<before>..HEAD <id> unattended` (if the
> Skill tool is unavailable, read `${CLAUDE_PLUGIN_ROOT}/skills/review-change/SKILL.md` and
> follow it exactly), and end with the lines that skill defines.

Log its `SCOPE:` and `RESULT:` lines as `review <id> …`. `RESULT: FAILED` doesn't stop the
run (the reviewer undid its own edits). The tree must be clean afterwards, as in section 3, step 5.

## 5. Failures

A failure never stops the run on its own; the attempt is saved and the run moves on.

1. **Save the attempt** if the tree has changes. Use the WIP branch name from
   `list-format.md` (append `-2`, `-3` if it exists):

   ```sh
   git switch -c workflow/failed/<list>-<id>
   git add -A
   git commit --no-verify -m "wip: failed attempt at <list> <id>" -m "<reason>"
   git switch <run branch>
   ```

   `--no-verify` only here: the attempt is broken by definition, and hooks would refuse to
   save it. The tree was clean before the item, so everything in it belongs to the attempt.
2. **Escalate once.** If `## Runs` names an escalation model, the item's model differs from
   it and the item wasn't escalated yet: run section 3, steps 2–4 again for this id with that model,
   appending to the prompt: "An earlier attempt with <model> failed: <reason>. Its changes
   are on branch `<branch>` (`git diff <run branch>...<branch>`); use them if helpful, but
   start from the current code." (If step 1 saved nothing, leave out the branch sentence.)
   If that succeeds, delete the saved branch (`git branch -D`) and continue as for `DONE`. If it fails, save that attempt as well (step 1).
3. **Record the failure.** `workflow-lists set <id> failed <last saved branch, or none>
   <model> "<reason>"` and commit the list file
   (`chore: record failed attempt at <id>`, no `Refs:` footer). Log
   `failed <id> <branch> — <reason>`, count up `consecutive_failures`, and stop the run when
   it reaches the limit from `## Runs` (something is probably wrong beyond a single item).

## 6. Items blocked during the run

If items were blocked during the run (`BLOCKED:` lines in the log) when `queue` runs out: run
section 2 again for just those ids (triage, ask, record), queue the decided ones and continue
with section 3. Do this once per item; an item blocked again after the user decided stays open
and goes into the report.

## 7. After the run

If **analyse after run** is `yes` for this kind of run and the run made commits:

1. Size the change: `git diff --shortstat <start>..HEAD -- . ':!*_PLAN.md' ':!*_IDEAS.md'
   ':!*_ANALYSIS.md'` (and `--stat` for the file names; don't read the code).
2. **Large** (above roughly 40 files or 8000 changed lines): the analysis agent can't start
   agents, so split here. Group the changed files into 3–8 parts by area (the scopes from
   `workflow-lists section USER_WORKFLOW.md Commits`, or top-level directories), then spawn
   one `general-purpose` subagent per part, in parallel (model `analysis`), description
   `Analyse part <n>`:

   > Read-only analysis; don't edit any project file. Read
   > `${CLAUDE_PLUGIN_ROOT}/skills/analyse-code/part.md` and follow it as a part agent for
   > <files> since <start>, `after-run`, with the output file `<run dir>/analysis/part-<n>.md`.

   Wait for all of them (`PART DONE` lines). `<parts>` is then
   ` parts=<run dir>/analysis`; for a small change it is empty.
3. Spawn one `general-purpose` subagent (model `analysis`), description
   `Analyse run changes`:

   > You are working in the repo at the current directory, unattended. Invoke the
   > `dev-workflow:analyse-code` skill with the argument
   > `since=<start> after-run<parts> unattended` (if the Skill tool is unavailable, read
   > `${CLAUDE_PLUGIN_ROOT}/skills/analyse-code/SKILL.md` and follow it exactly), and end with
   > the lines that skill defines.

   Log its `SCOPE:` and `RESULT:` lines, then remove `<run dir>/analysis`.

## 8. Final report

Log `end`, then read the log once and build the report from it. Short:

- the commits made (`git log --oneline <start>..HEAD`), and the run branch if one was created
  (to be merged by the user);
- items removed as already done (`gone`), with the reason;
- items still open: blocked (with the decision needed), skipped by the user, on hold;
- failed items with their WIP branches and reasons;
- the logged `DEFAULTS:` the user may want to tune, and what couldn't be verified;
- reviews that made changes;
- the after-run analysis (new findings, suggesting `/fix-all-findings`);
- the logged `SCOPE:` lines (scopes to add to `USER_WORKFLOW.md`);
- missing `## Runs` settings, if defaults were used;
- the `error` lines of a final `workflow-lists check <lists>`, if any (a single-item agent
  left a list broken).

Remind the user that nothing was pushed.
