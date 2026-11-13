# Working off a whole list: shared run protocol

Read by the loop skills (`fix-all-findings`, `implement-all-features`). They name the lists,
the kind of work (`fixes` / `features`), their own triage cases and how deep triage looks at
the code; everything else is here.

A run has two halves. **You** do the first: triage and every decision the user has to make,
all before anything is built. **`workflow-lists run go`** does the second, unattended: it
starts one fresh headless agent (`claude -p`) per item with the model, effort, turn and cost
limits from `## Runs`, books each answer, reviews, reworks, retries and escalates, saves failed
attempts on `workflow/failed/<item>` branches, runs the closing analysis, writes one stats
line per agent, and prints the report. You never read or change project code, and you never
run two drivers at once.

All questions come **before** the unattended half. After that the run needs the user only for
the rare item that turns out to need a decision mid-run (section 4), which pauses the driver.

## 1. Start

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. `workflow-lists run start <kind> [--user-only] [--max <n>] [--skip <ids>] --footer "<the
   attribution lines from the system reminder>"`, with the skill's arguments (a maximum
   number of items → `--max`, `skip=` → `--skip`) and no `--footer` if there are no
   attribution lines. It checks that the tree is clean (`DIRTY`: show the user the changes and
   stop; each item commits), reads `## Runs`, creates the run branch if the settings ask for
   one, and prints the settings (mention missing ones in one line; the defaults are used).
   - `RESUMED`: an interrupted run continues. Say so in one line. If it printed a queue that
     isn't empty, go straight to section 3; otherwise do section 2.
3. `workflow-lists check <lists>`: errors in the lists go into the final report; the run goes
   ahead unless they make items unreadable.

## 2. Triage and decisions

1. **Triage.** `workflow-lists triage <lists> --skip <skipped ids>` (in section 4 with
   `--only <ids>`). It prints `ORDER:` (the order of work: suggested order, what others build
   on, severity, size; the user's findings first), then one line per item: `READY` (workable
   as written), `FAILED` (has a `_Failed:_` stamp: a DECIDE item with the fixed question
   below), `CHECK` (needs a look at the code, with the reason), and `HELD` / `SKIPPED` ids.
   Nothing open besides held items: say so, `workflow-lists run end <kind>`, and stop. Only
   if there are `CHECK` items, spawn one `Plan` subagent (read-only, model `triage` from the
   settings `run start` printed, omitted for `inherit`), description `Triage <kind>`, with
   this prompt, adding the skill's own triage cases:

   > Read sections A and B of `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/work-item.md`,
   > `CLAUDE.md`, and of `USER_WORKFLOW.md` only
   > `workflow-lists section USER_WORKFLOW.md "Design docs" "Ask before" "Hands-off"`.
   > Classify only these items, read with `workflow-lists show <ids>`: <the CHECK lines>.
   > Check each against the current code only as deeply as needed to classify it (<skill's
   > triage depth>): READY (workable unattended, including items whose **Decisions:** cover
   > their open questions), DECIDE (needs the user per section B, plus: <skill's cases>), or
   > GONE (already done or obsolete). Don't plan READY items. For each DECIDE item, work out
   > the approach and the concrete decisions the user must make. Answer only in this format,
   > one line per item:
   >
   > `READY: <id>`
   > `DECIDE: <id> | <title> — <the planned approach in one or two lines>`
   > `  Q: <question> | <recommended option> | <option> [| <option>]` (one to three per item)
   > `GONE: <id> — <why>`

   `${CLAUDE_PLUGIN_ROOT}` is not substituted in this file: replace it with the plugin root
   (the directory two levels above this file) before sending the prompt.

   For a `FAILED` item the only question is `How to continue with <id>?` with the options
   `Retry from scratch`, `Continue from the WIP branch`, `Put on hold`, `Drop it`
   (`Continue from the WIP branch` only if it names a branch); show its reason.

2. **Ask.** If there are DECIDE items, show the user each one in two lines (id, title,
   approach), then ask their questions with `AskUserQuestion`, up to four questions per call,
   recommended option first. Add `Skip this run` as the last option of each item's first
   question; if it already has four options (a `FAILED` item), `Drop it` gives way, still
   reachable through "Other". Ask nothing else: no run confirmation, no model choice.
3. **Record.** For every DECIDE and GONE item:
   - `Skip this run`: `workflow-lists run skip <kind> <id>`.
   - `Put on hold`: `workflow-lists set <id> hold`, then `run skip`.
   - `Drop it`: `workflow-lists delete <id>`.
   - Every GONE item (no question asked): `workflow-lists delete <id>` and
     `workflow-lists run log <kind> "gone <id> — <why>"`. Triage already checked the code,
     so no agent is spent on it; the final report lists them, so the user can restore one
     from git if triage was wrong.
   - Otherwise `workflow-lists set <id> decisions "<the answers, in the user's words where
     they wrote their own>"`. For a `FAILED` item it moves the stamp (branch, model, reason)
     into the decision line.

   Fix every `error` the commands print. Then `workflow-lists run commit <kind> "chore:
   record run decisions for <ids>"` (or `chore: remove done items <ids>`, both parts when
   both apply).
4. **Queue:** `workflow-lists run queue <kind> <ids>` with `ORDER` minus skipped, held,
   dropped and gone ids, each as `<id>:<size>` where triage gave a size (`--append` in
   section 4). Tell the user in one line how many items the run will do, the models and
   limits it uses, and that it now runs unattended.

## 3. Drive

Run `workflow-lists run go <kind>` with the Bash tool **in the background**
(`run_in_background`; it can take hours), tell the user it is running, and end your turn. The
user can also run the same command from their shell (`workflow-lists` is on the plugin's
`bin/` path, or they run it by its full path), and `workflow-lists run go <kind> --auto`
starts a run without this skill, queuing only the items triage marks `READY`.

The driver prints one block per item (`NEXT:` line, the agent's result lines, the action
taken with turns and cost) and ends with `REPORT:`. Don't poll it and don't predict its
result. When its completion arrives, read the output from the end:

- Exit code 0 and a `REPORT:`: section 5.
- Exit code 3 and `REDECIDE: <ids>`: section 4.
- Exit code 1 and `STOP: …` before the report: a dirty tree after a reported success, or too
  many failures in a row. Section 5, with the reason first.
- No JSON from `claude` (the item lines say `claude exited … without JSON output`): the
  headless agents can't run here; show the error and stop. The usual causes: `claude` not on
  the `PATH` of the sandbox, a permission mode the sandbox refuses, or no login.

If a `run go` has to be stopped by hand (the Bash tool's `TaskStop`), the next `run start`
resumes the run; an item that was mid-way is recorded as failed with its attempt on a branch.

## 4. Items blocked during the run

`REDECIDE: <ids>` lists items that agents blocked during the run. Do section 2 for just those
ids (`triage --only <ids>`, ask, record), `workflow-lists run queue <kind> --append` the
decided ones and go back to section 3. The driver offers each item once; an item blocked
again stays open and goes into the report.

## 5. After the run

The driver already ran the closing analysis (if `## Runs` asks for it; large changes are split
across parallel read-only part agents) and printed the report: commits, done, gone, blocked
(with the decision needed), skipped, held, failed (with branches), defaults, what couldn't be
verified, the checks the item agents ran, notes from the agents, reviews that changed something
or asked for rework, the analysis result, scopes to add, missing settings, list errors and the
cost. Give it to the user in short form: the commits and run branch first, then what needs
their attention (blocked, failed, unverified, notes, defaults to tune), the cost, and
`/fix-all-findings` if the analysis found something. Say that nothing was pushed. Point them to
`workflow-lists stats` for the numbers of this and earlier runs.
