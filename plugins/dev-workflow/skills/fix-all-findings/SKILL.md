---
name: fix-all-findings
description: Work off CLAUDE_ANALYSIS.md and the user's USER_ANALYSIS.md completely, one finding per fresh subagent: each agent runs /fix-next-finding unattended (fix, verify, delete entry, commit), then the next one starts, until nothing fixable is left; items needing the user's decision come last, with only the decision asked. Use when the user asks to fix all findings, work off the whole analysis list, or loop /fix-next-finding. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Fix all analysis findings

The user asked for this loop, so spawning subagents is intended here. You are the
**orchestrator**: you don't fix anything yourself and you don't read the code. Each finding is
fixed by a fresh `general-purpose` subagent, so every fix starts with an empty context and
only the short result lines collect here. Subagents can't start further agents, so the loop
runs in this session.

Both lists are worked off: `CLAUDE_ANALYSIS.md` and the user's `USER_ANALYSIS.md`.
`fix-next-finding` decides which comes next; ids of the user's items carry a `U` prefix
(`U1.1`) in arguments, `skip=` lists and `RESULT:` lines.

Optional argument: a maximum number of fixes (e.g. `5`), and/or `skip=<ids>` for findings to
leave alone from the start, and/or `user-only` (used by `/fix-all-user-findings`): then only
`USER_ANALYSIS.md` is worked off. Count only its headings before the loop, add `user-only` to
every subagent's skill argument (`unattended user-only skip=<ids>`, and `<id> unattended
user-only` in phase 2), and in phase 2 take only `U` items.

## Before the loop

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. `git status --short` must be clean. If it isn't, show the user the changes and stop: each
   iteration commits, and unrelated changes would get in the way.
3. Read only the headings of both files (`grep -n '^###' CLAUDE_ANALYSIS.md USER_ANALYSIS.md`)
   to tell the user how many findings are open in each (and how many are on `[HOLD]`, which
   the loop leaves alone). If there are none besides held ones, say so and stop.
4. Start with `skip` = the ids from the argument, `blocked` = empty, `fixed` = empty,
   `scopes` = empty, and note `git rev-parse --short HEAD` as the start.

## Each iteration

1. Spawn one subagent (`subagent_type: general-purpose`, not `fork`), description
   `Fix next finding`, with this prompt (fill in the skip list):

   > You are working in the repo at the current directory, unattended: nobody will answer
   > questions. Invoke the `dev-workflow:fix-next-finding` skill with the argument `unattended
   > skip=<ids>` (if the Skill tool is unavailable, read
   > `${CLAUDE_PLUGIN_ROOT}/skills/fix-next-finding/SKILL.md` and follow it exactly). Fix exactly one
   > finding, verify it, update its analysis file, commit, and end with the `BLOCKED:`,
   > `SCOPE:` and final `RESULT:` lines that skill defines.

2. Wait for its completion notification. Don't start the next agent before it is done (they
   would edit the same files). Don't poll and don't predict its result.
3. Read the `BLOCKED:`, `SCOPE:` and `RESULT:` lines from its report. Add the blocked ids to
   `blocked` and `skip`, and the `SCOPE:` lines to `scopes`. Then:
   - `FIXED <id> <commit>`: add to `fixed`. Give the user one line (`✓ 1.5 Money format`).
   - `PARTIAL <id> <commit>`: add to `fixed` as partial. If the same id comes back partial a
     second time, add it to `skip` so the loop can't spin on it.
   - `NOTHING_LEFT`: stop the loop and go to the final report.
   - `BLOCKED`: the unattended phase is over; go to "Phase 2".
   - `FAILED <id>`, or no `RESULT:` line at all: stop the loop. The agent's changes are in
     the working tree; don't revert them, leave them for the user.
4. Check `git status --short` is clean (`FIXED`/`PARTIAL` must have committed). If not, stop
   and report what is left.
5. Stop when the maximum number of fixes is reached; otherwise start the next iteration.

## Phase 2: items that need the user

Only now, after everything fixable unattended is done, take the `blocked` items (not the ones
the user put in `skip=`). Go through them one at a time, the user's `U` items first, each
file in the order of its section 4:

1. **Plan.** Spawn a `Plan` subagent (read-only) with: "Read CLAUDE.md, USER_WORKFLOW.md and
   finding `<id>` in <CLAUDE_ANALYSIS.md, or USER_ANALYSIS.md for a `U` id, naming the number
   without the `U`>, check it against the current code, and propose how to fix it: the
   approach, the files touched, risks (persisted formats, load-bearing names, behaviour
   changes, anything under 'Ask before' in USER_WORKFLOW.md), and the concrete decisions the
   user has to make, each with 2–4 options and a recommendation. If the item is already fixed
   or obsolete, say so." Wait for it.
2. **Ask.** Show the user the plan in a few lines and ask the decisions with
   `AskUserQuestion` (recommended option first). Always include the choice to skip this item.
   If the plan says the item is obsolete, ask whether to delete the entry.
3. **Fix.** If approved, spawn a `general-purpose` subagent with the same prompt as in the
   loop, but with argument `<id> unattended` and the user's decision appended verbatim
   ("The user's decision for <id>: …"). Handle its result lines like in the loop. On
   `FAILED`, stop and report.
4. Skipped items stay in their file. Move on to the next blocked item.

A big refactor can change the code that later findings refer to. After each phase-2 fix,
run one more unattended loop iteration if the agent's report mentions that something else
became fixable.

## Final report

Short: the commits made (`git log --oneline <start>..HEAD`), findings still open (and which
of them the user chose to skip in phase 2), the collected `scopes` (commits without a scope,
with the suggested scopes to add to `USER_WORKFLOW.md`), and the failure output if the loop
ended on `FAILED`. Remind the user that nothing was pushed.
