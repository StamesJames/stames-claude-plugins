# Working off a whole list: shared run protocol

Read by the loop skills (`fix-all-findings`, `implement-all-features`). They name the lists,
the single-item skill, the kind of work (`fixes` / `features`) and their own triage cases;
everything else is here.

The user asked for this run, so spawning subagents is intended. You are the
**orchestrator**: you never read or change project code. Each item is built by a fresh
subagent, so every item starts with an empty context and only short result lines collect
here. Subagents can't start further agents, so the loop runs in this session. The only files
you edit are the list files (decisions and failure stamps).

All questions come **before** the long unattended part (step 2). After that the run doesn't
need the user, apart from the rare item that turns out to need a decision mid-run (step 6).
All state that matters is written to the list files, so an interrupted run can simply be
started again.

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
4. Count the open items with `workflow-lists summary <lists>` (don't read the lists for
   this): its `TOTAL:` line gives open and held items, its item lines the ids and line
   numbers. None open besides held ones: say so and stop. Run `workflow-lists check <lists>`
   too; errors in the lists go into the final report, and the run goes ahead unless they
   make items unreadable.
5. Note the branch (`git branch --show-current`) and `git rev-parse --short HEAD` as the
   start. If **run branch** is `yes` and the current branch is the default branch (`main`,
   `master`, or what `git symbolic-ref --short refs/remotes/origin/HEAD` names), create and
   switch to `workflow/<YYYY-MM-DD>-<kind>` (append `-2`, `-3` if it exists). The run commits
   to the current branch from here on; call it the **run branch**.
6. State: `queue`, `skip` (ids from the argument), `blocked`, `done`, `defaults`, `scopes`,
   `reviews`, `failed`, `consecutive_failures = 0`.

**Models.** Every subagent gets the model for its kind of work from `## Runs` (`triage`,
`fixes`, `features`, `review`, `analysis`); an item's `**Model:**` line overrides it. Pass it
as the Agent tool's `model` (omit it for `inherit`), and a reasoning level, if one is given,
as `effort`.

## 2. Triage and decisions

1. **Triage.** Spawn one `Plan` subagent (read-only, model `triage`), description
   `Triage <kind>`, with this prompt, adding the skill's own triage cases and lists:

   > Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`, sections A and B of
   > `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/work-item.md`, `CLAUDE.md`,
   > `USER_WORKFLOW.md` and <lists>. Consider every open item <of the given ids> that is not
   > tagged [HOLD], not in `skip=<ids>`, and not in an `## Ideas` section. For each, check it
   > against the current code only as deeply as needed to classify it: READY (workable
   > unattended, including items whose **Decisions:** cover their open questions), DECIDE
   > (needs the user per section B, plus: <skill's cases>; every item with a `_Failed:_`
   > stamp), or GONE (already done or obsolete). Don't plan READY items. For each DECIDE
   > item, work out the approach and the concrete decisions the user must make. Answer only
   > in this format:
   >
   > `ORDER: <ids>` — every READY and DECIDE item, in the order to work them (the lists'
   > suggested order, then items others build on, then severity, then smaller first)
   > `READY: <id> | size <S/M/L/?> | model <the item's **Model:** line, or default>`
   > `DECIDE: <id> | size | model | <title> — <the planned approach in one or two lines>`
   > `  Q: <question> | <recommended option> | <option> [| <option>]` (one to three per item)
   > `GONE: <id> — <why>`
   >
   > For a `_Failed:_` item the only question is `How to continue with <id>?` with the options
   > `Retry from scratch`, `Continue from the WIP branch`, `Put on hold`, `Drop it`.

2. **Ask.** If there are DECIDE items, show the user each one in two lines (id, title,
   approach), then ask their questions with `AskUserQuestion`, up to four questions per call,
   recommended option first. Add `Skip this run` as the last option of each item's first
   question (dropping the weakest option if it already has four). Ask nothing else: no run
   confirmation, no model choice.
3. **Record.** For every DECIDE item, edit its entry in the list file:
   - `Skip this run`: no edit; add the id to `skip`.
   - `Put on hold`: add `[HOLD]` after the severity; add the id to `skip`.
   - `Drop it`: delete the entry with its mentions (as `list-format.md` describes) and the
     branches it names.
   - Otherwise add `**Decisions:** <the answers, in the user's words where they wrote their
     own> (<date>)` at the end of the entry. For a `_Failed:_` item, replace the stamp: the
     decision line names the branch and the earlier failure reason
     (`Retry from scratch; earlier attempt on `workflow/failed/...` failed: <reason>`).

   Run `workflow-lists check <the lists you edited>` and fix every `error` in your edits.
   Commit the list files you edited (by explicit path; type `chore`,
   `chore: record run decisions for <ids>`, no `Refs:` footer, scope per `list-format.md`).
4. **Queue:** GONE items first (the single-item skill confirms and deletes them), then
   `ORDER` without skipped ids. Tell the user in one line how many items the run will do and
   that it now runs unattended.

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
4. Read the `BLOCKED:`, `DEFAULTS:`, `SCOPE:` and `RESULT:` lines. Collect `DEFAULTS:` and
   `SCOPE:` lines. Then:
   - `DONE <id> <commit>`: add to `done`, reset `consecutive_failures`, run the review
     (step 4). Give the user one line (`✓ F2 Pause menu sound`) plus the defaults chosen.
   - `PARTIAL <id> <commit>`: as `DONE`, marked partial; put the id at the end of `queue`
     once. A second `PARTIAL` for the same id: add it to `skip`.
   - `BLOCKED <id>`: add to `blocked` with the reason from its `BLOCKED:` line.
   - `NOTHING_LEFT`: the item was gone; carry on.
   - `FAILED <id>`, or no `RESULT:` line: step 5.
5. Except after a failure, `git status --short` must be clean. If it isn't, stop and report
   what is left.
6. Stop when the maximum number of items from the argument is reached. When `queue` is empty,
   go to step 6.

## 4. Review

Decide from `## Runs` → **fresh review** whether the item gets one: its triage size, and the
changed lines of `git diff --shortstat <before>..HEAD -- . ':!*_PLAN.md' ':!*_ANALYSIS.md'`.
If it does, spawn a `general-purpose` subagent (model `review`), description `Review <id>`:

> You are working in the repo at the current directory, unattended. Invoke the
> `dev-workflow:review-change` skill with the argument `<before>..HEAD <id> unattended` (if the
> Skill tool is unavailable, read `${CLAUDE_PLUGIN_ROOT}/skills/review-change/SKILL.md` and
> follow it exactly), and end with the lines that skill defines.

`RESULT: REVIEWED <commit>` or `RESULT: CLEAN` go into `reviews`; `RESULT: FAILED` is noted
for the report but doesn't stop the run (the reviewer undid its own edits). The tree must be
clean afterwards, as in step 3.5.

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
   it and the item wasn't escalated yet: run step 3 again for this id with that model,
   appending to the prompt: "An earlier attempt with <model> failed: <reason>. Its changes
   are on branch `<branch>` (`git diff <run branch>...<branch>`); use them if helpful, but
   start from the current code." If that succeeds, delete the saved branch (`git branch -D`)
   and continue as for `DONE`. If it fails, save that attempt as well (1).
3. **Record the failure.** Add the `_Failed:_` stamp from `list-format.md` to the entry
   (naming the last saved branch, the model and the reason; time from
   `workflow-lists stamp`) and commit the list file
   (`chore: record failed attempt at <id>`, no `Refs:` footer). Add the id to `failed` and
   `skip`, count up `consecutive_failures`, and stop the run when it reaches the limit from
   `## Runs` (something is probably wrong beyond a single item).

## 6. Items blocked during the run

If `blocked` is not empty when `queue` runs out: run step 2 again for just those ids (triage,
ask, record), queue the decided ones and continue with step 3. Do this once per item; an item
blocked again after the user decided stays open and goes into the report.

## 7. After the run

If **analyse after run** is `yes` for this kind of run and the run made commits, spawn a
`general-purpose` subagent (model `analysis`), description `Analyse run changes`:

> You are working in the repo at the current directory, unattended. Invoke the
> `dev-workflow:analyse-code` skill with the argument `since=<start> unattended` (if the Skill
> tool is unavailable, read `${CLAUDE_PLUGIN_ROOT}/skills/analyse-code/SKILL.md` and follow it
> exactly), and end with the lines that skill defines.

## 8. Final report

Short:

- the commits made (`git log --oneline <start>..HEAD`), and the run branch if one was created
  (to be merged by the user);
- items still open: blocked (with the decision needed), skipped by the user, on hold;
- failed items with their WIP branches and reasons;
- the collected `DEFAULTS:` the user may want to tune, and what couldn't be verified;
- reviews that made changes;
- the after-run analysis (new findings, suggesting `/fix-all-findings`);
- the collected `SCOPE:` lines (scopes to add to `USER_WORKFLOW.md`);
- missing `## Runs` settings, if defaults were used;
- the `error` lines of a final `workflow-lists check <lists>`, if any (a single-item agent
  left a list broken).

Remind the user that nothing was pushed.
