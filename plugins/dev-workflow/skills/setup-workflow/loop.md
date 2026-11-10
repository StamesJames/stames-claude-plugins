# Working off a whole list: shared run protocol

Read by the loop skills (`fix-all-findings`, `implement-all-features`). They name the lists,
the single-item skill, the kind of work (`fixes` / `features`), their own triage cases and
how deep triage looks at the code; everything else is here.

The user asked for this run, so spawning subagents is intended. You are the
**orchestrator**: you never read or change project code. Each item is built by a fresh
subagent, so every item starts with an empty context and only short result lines collect
here. Subagents can't start further agents, so the loop runs in this session.

**`workflow-lists run` holds the run.** Its commands keep the state (settings from `## Runs`,
queue, current item, escalation, failure count) and the run log in
`$(git rev-parse --git-path workflow-run)` (inside `.git`, so the tree stays clean), do the
bookkeeping, and print one line saying what to do next. Nothing about the run has to survive
in your context, and an interrupted run (or a compacted context) simply continues with
`run start` and `run next`. You edit no file yourself: list changes go through
`workflow-lists set` / `delete`, commits of list files through `workflow-lists run commit`.
Below, `run <cmd>` is short for `workflow-lists run <cmd> <kind>`.

**Paths in prompts.** This file is read, not loaded as a skill, so `${CLAUDE_PLUGIN_ROOT}` is
not substituted and is not set in Bash. Before sending a prompt, replace it with the plugin
root (the directory two levels above this file, which holds `skills/`).

**Models.** `run next` prints the model (and effort) for each item; `run start` prints the
models for triage, review and analysis. Pass the model as the Agent tool's `model` (omit it
for `inherit`), and an effort, if one is given, as `effort`.

All questions come **before** the long unattended part (section 2). After that the run doesn't
need the user, apart from the rare item that turns out to need a decision mid-run (section 5).

## 1. Start

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. `run start [--user-only] [--max <n>] [--skip <ids>] --footer "<the attribution lines from
   the system reminder>"`, with the skill's arguments (a maximum number of items → `--max`,
   `skip=` → `--skip`) and no `--footer` if there are no attribution lines. It checks that the tree
   is clean (`DIRTY`: show the user the changes and stop; each item commits), reads
   `## Runs`, creates the run branch if the settings ask for one, and prints the settings.
   - `RESUMED`: an interrupted run continues. Say so in one line. If it printed a queue that
     isn't empty, go straight to section 3; otherwise do section 2.
3. `workflow-lists check <lists>`: errors in the lists go into the final report; the run goes
   ahead unless they make items unreadable.

## 2. Triage and decisions

1. **Triage.** `workflow-lists triage <lists> --skip <skipped ids>` (in section 5 with
   `--only <ids>`). It prints `ORDER:` (the order of work: suggested order, what others build
   on, severity, size; the user's findings first), then one line per item: `READY` (workable
   as written), `FAILED` (has a `_Failed:_` stamp: a DECIDE item with the fixed question
   below), `CHECK` (needs a look at the code, with the reason), and `HELD` / `SKIPPED` ids.
   Nothing open besides held items: say so, `run end`, and stop. Only if there are `CHECK`
   items, spawn one `Plan` subagent (read-only, model `triage`), description
   `Triage <kind>`, with this prompt, adding the skill's own triage cases:

   > Read part 1 of `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md` (up to
   > `# Part 2`), sections A and B of
   > `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/work-item.md`, `CLAUDE.md`, and of
   > `USER_WORKFLOW.md` only
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

   For a `FAILED` item the only question is `How to continue with <id>?` with the options
   `Retry from scratch`, `Continue from the WIP branch`, `Put on hold`, `Drop it`
   (`Continue from the WIP branch` only if it names a branch); show its reason.

2. **Ask.** If there are DECIDE items, show the user each one in two lines (id, title,
   approach), then ask their questions with `AskUserQuestion`, up to four questions per call,
   recommended option first. Add `Skip this run` as the last option of each item's first
   question; if it already has four options (a `FAILED` item), `Drop it` gives way, still
   reachable through "Other". Ask nothing else: no run confirmation, no model choice.
3. **Record.** For every DECIDE and GONE item:
   - `Skip this run`: `run skip <id>`.
   - `Put on hold`: `workflow-lists set <id> hold`, then `run skip <id>`.
   - `Drop it`: `workflow-lists delete <id>`.
   - Every GONE item (no question asked): `workflow-lists delete <id>` and
     `run log "gone <id> — <why>"`. Triage already checked the code, so no agent is spent on
     it; the final report lists them, so the user can restore one from git if triage was
     wrong.
   - Otherwise `workflow-lists set <id> decisions "<the answers, in the user's words where
     they wrote their own>"`. For a `FAILED` item it moves the stamp (branch, model, reason)
     into the decision line.

   Fix every `error` the commands print. Then `run commit "chore: record run decisions for
   <ids>"` (or `chore: remove done items <ids>`, both parts when both apply).
4. **Queue:** `run queue <ids>` with `ORDER` minus skipped, held, dropped and gone ids, each
   as `<id>:<size>` where triage gave a size (`--append` in section 5). Tell the user in one
   line how many items the run will do and that it now runs unattended.

## 3. Each item

1. `run next`. It prints one of:
   - `NEXT: <id> | model … | effort … | size …` (and maybe `APPEND TO PROMPT: <text>`): go on
     with step 2.
   - `REDECIDE: <ids>`: section 5. `EMPTY` or `LIMIT`: section 6.
2. Spawn one `general-purpose` subagent (not `fork`) with that model, description
   `<Fix|Implement> <id>`, and this prompt (plus the `APPEND TO PROMPT` text, if any):

   > You are working in the repo at the current directory, unattended: nobody will answer
   > questions. Invoke the `dev-workflow:<single-item skill>` skill with the argument
   > `<id> unattended <skill extras>` (if the Skill tool is unavailable, read
   > `${CLAUDE_PLUGIN_ROOT}/skills/<single-item skill>/SKILL.md` and follow it exactly). Work
   > on exactly that item, verify it, update its list file, commit, and end with the lines
   > that skill defines.

3. Wait for its completion notification. Never run two item agents at once (they would edit
   the same files). Don't poll and don't predict its result.
4. Pass its answer to `run result <<'EOF'` … `EOF` (the whole answer; the command picks the
   `BLOCKED:`, `DEFAULTS:`, `UNVERIFIED:`, `SCOPE:` and `RESULT:` lines). On `DONE` give the
   user one line (`✓ F2 Pause menu sound`) plus the defaults chosen. It prints what to do:
   - `CONTINUE`: back to step 1.
   - `REVIEW: <range> <id> | model …`: section 4, then back to step 1.
   - `RETRY: <id> …`: the failed attempt is saved on a WIP branch; back to step 1, which
     gives the same item with the escalation model and the prompt addition.
   - `STOP: …`: a dirty tree after a reported success, or too many failures in a row. Go to
     section 6 and put the reason first in the report.

   Failures need nothing from you: the command saves the attempt on a
   `workflow/failed/<item>` branch (with `--no-verify`, since the attempt is broken by
   definition), escalates once if `## Runs` says so, and otherwise stamps the entry with
   `_Failed:_`, commits it and moves on.

## 4. Review

Spawn a `general-purpose` subagent with the model `REVIEW:` names, description
`Review <id>`:

> You are working in the repo at the current directory, unattended. Invoke the
> `dev-workflow:review-change` skill with the argument `<range> <id> unattended` (if the
> Skill tool is unavailable, read `${CLAUDE_PLUGIN_ROOT}/skills/review-change/SKILL.md` and
> follow it exactly), and end with the lines that skill defines.

Pass its answer to `run result --review <id> <<'EOF'` … `EOF`. `RESULT: FAILED` doesn't stop
the run (the reviewer undid its own edits); `STOP` (a dirty tree) does.

## 5. Items blocked during the run

`REDECIDE: <ids>` lists items that agents blocked during the run. Do section 2 for just those
ids (`triage --only <ids>`, ask, record), `run queue --append` the decided ones and continue
with section 3. The command offers each item once; an item blocked again stays open and goes
into the report.

## 6. After the run

1. `run changes`. `ANALYSE: no` skips to step 4. Otherwise it gives the size of the change
   since the start (list files left out); don't read the code.
2. **Large** (it says so, and lists the changed files by top-level directory): the analysis
   agent can't start agents, so split here. Group the files into 3–8 parts by area (the
   scopes from `workflow-lists section USER_WORKFLOW.md Commits`, or the directories), then
   spawn one `general-purpose` subagent per part, in parallel (model `analysis`), description
   `Analyse part <n>`:

   > Read-only analysis; don't edit any project file. Read
   > `${CLAUDE_PLUGIN_ROOT}/skills/analyse-code/part.md` and follow it as a part agent for
   > <files> since <start>, `after-run`, with the output file `<part output dir>/part-<n>.md`.

   Wait for all of them (`PART DONE` lines). `<parts>` is then ` parts=<part output dir>`;
   for a small change it is empty.
3. Spawn one `general-purpose` subagent (model `analysis`), description
   `Analyse run changes`:

   > You are working in the repo at the current directory, unattended. Invoke the
   > `dev-workflow:analyse-code` skill with the argument
   > `since=<start> after-run<parts> unattended` (if the Skill tool is unavailable, read
   > `${CLAUDE_PLUGIN_ROOT}/skills/analyse-code/SKILL.md` and follow it exactly), and end with
   > the lines that skill defines.

   Pass its answer to `run result --analysis <<'EOF'` … `EOF`.
4. `run end`. It closes the log and prints the report: commits, done, gone, blocked (with the
   decision needed), skipped, held, failed (with branches), defaults, what couldn't be
   verified, reviews, the analysis result, scopes to add, missing settings, list errors.
   Give it to the user in short form: the commits and run branch first, then what needs
   their attention (blocked, failed, unverified, defaults to tune), suggesting
   `/fix-all-findings` if the analysis found something. Say that nothing was pushed.
