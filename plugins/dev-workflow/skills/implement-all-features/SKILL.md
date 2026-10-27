---
name: implement-all-features
description: Work off USER_PLAN.md completely (never CLAUDE_PLAN.md), one item per fresh subagent: each agent runs /implement-next-feature unattended (implement, verify, delete entry, commit), then the next one starts, until nothing implementable is left; items needing the user's decision (ideas, ambiguous or big items) come last, with only the decision asked. Use when the user asks to implement all of their plan, work off USER_PLAN.md, or loop /implement-next-feature. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Implement all user plan items

The user asked for this loop, so spawning subagents is intended here. You are the
**orchestrator**: you don't implement anything yourself and you don't read the code. Each item
is implemented by a fresh `general-purpose` subagent, so every item starts with an empty
context and only the short result lines collect here. Subagents can't start further agents,
so the loop runs in this session.

**Only `USER_PLAN.md`**, unless the argument contains `claude-plan` (passed only by
`/implement-all-claude-features`). Without it, `CLAUDE_PLAN.md` is never touched; its items
are pitched one at a time with `/pitch-feature`.

**`claude-plan` mode** works off `CLAUDE_PLAN.md` instead: read `CLAUDE_PLAN.md` wherever this
skill says `USER_PLAN.md`, pass `claude-plan` on to every `implement-next-feature` subagent
(after `unattended`), and treat `[CONFLICT]` items like ideas (phase 2, after the concrete
features). The run needs the user's explicit confirmation first (step 4 below).

Optional argument: a maximum number of items (e.g. `3`), and/or `skip=<ids>` for items to
leave alone from the start.

## Before the loop

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. `git status --short` must be clean. If it isn't, show the user the changes and stop: each
   iteration commits, and unrelated changes would get in the way.
3. Read only the headings of `USER_PLAN.md` (`grep -n '^##' USER_PLAN.md`) to tell the user
   how many items are open (concrete features and ideas, and how many are on `[HOLD]`). If
   there are none besides held ones, say so and stop.
4. **`claude-plan` mode only: confirm.** These are Claude's own ideas that the user hasn't
   approved one by one, so ask before building anything. Show the open item headings (id,
   severity, tags, name; held items marked as left alone), say that the concrete features
   will be implemented and committed one after another without further questions while ideas
   and `[CONFLICT]` items are asked at the end, and mention in one line how many items are
   open in `USER_PLAN.md` (not part of this run). Then ask with `AskUserQuestion`:
   - `Implement all` — run the loop as described.
   - `Only some` — the user names the ids to leave out (added to `skip`) or a maximum
     number; then run the loop.
   - `Pitch one instead` — stop and suggest `/pitch-feature`.
   - `Cancel` — stop.

   Only an explicit `Implement all` or `Only some` starts the loop. Never skip this question,
   not even when the argument already has a maximum or `skip=`.
5. Start with `skip` = the ids from the argument, `blocked` = empty, `done` = empty,
   `defaults` = empty, `scopes` = empty, and note `git rev-parse --short HEAD` as the start.

## Each iteration

1. Spawn one subagent (`subagent_type: general-purpose`, not `fork`), description
   `Implement next feature`, with this prompt (fill in the skip list):

   > You are working in the repo at the current directory, unattended: nobody will answer
   > questions. Invoke the `dev-workflow:implement-next-feature` skill with the argument `unattended
   > skip=<ids>` (`unattended claude-plan skip=<ids>` in `claude-plan` mode, and
   > `CLAUDE_PLAN.md` instead of `USER_PLAN.md` below) (if the Skill tool is unavailable, read
   > `${CLAUDE_PLUGIN_ROOT}/skills/implement-next-feature/SKILL.md` and follow it exactly). Implement
   > exactly one item from USER_PLAN.md, verify it, update USER_PLAN.md, commit, and end with
   > the `BLOCKED:`, `DEFAULTS:`, `SCOPE:` and final `RESULT:` lines that skill defines.

2. Wait for its completion notification. Don't start the next agent before it is done (they
   would edit the same files). Don't poll and don't predict its result.
3. Read the `BLOCKED:`, `DEFAULTS:`, `SCOPE:` and `RESULT:` lines from its report. Add the
   blocked ids to `blocked` and `skip`, and the others to `defaults` and `scopes`. Then:
   - `DONE <id> <commit>`: add to `done`. Give the user one line (`✓ F2 Pause menu sound`),
     plus the defaults the agent chose if it listed any.
   - `PARTIAL <id> <commit>`: add to `done` as partial. If the same id comes back partial a
     second time, add it to `skip` so the loop can't spin on it.
   - `NOTHING_LEFT`: stop the loop and go to the final report.
   - `BLOCKED`: the unattended phase is over; go to "Phase 2".
   - `FAILED <id>`, or no `RESULT:` line at all: stop the loop. The agent's changes are in
     the working tree; don't revert them, leave them for the user.
4. Check `git status --short` is clean (`DONE`/`PARTIAL` must have committed). If not, stop
   and report what is left.
5. Stop when the maximum number of items is reached; otherwise start the next iteration.

## Phase 2: items that need the user

Only now, after everything implementable unattended is done, take the `blocked` items (not the
ones the user put in `skip=`). Go through them one at a time, concrete features before ideas,
in the order of `USER_PLAN.md`:

1. **Plan.** Spawn a `Plan` subagent (read-only) with: "Read CLAUDE.md, USER_WORKFLOW.md, the
   design docs it names and item `<id>` in USER_PLAN.md, check it against the current code,
   and propose how to implement it: the approach, the files touched, risks (persisted
   formats, load-bearing names, behaviour changes, conflicts with the design docs, anything
   under 'Ask before' in USER_WORKFLOW.md), and the concrete decisions the user has to make,
   each with 2–4 options and a recommendation. If the item is already done or obsolete, say
   so." Wait for it.
2. **Ask.** Show the user the plan in a few lines and ask the decisions with
   `AskUserQuestion` (recommended option first). Always include the choice to skip this item.
   If the plan says the item is obsolete, ask whether to delete the entry. In `claude-plan`
   mode, for a `[CONFLICT]` item say what it contradicts, and also offer `Drop it`: remove the
   entry and add a line to `## Dropped` as `/pitch-feature` describes (asking for a reason in
   one short question if the user gave none).
3. **Implement.** If approved, spawn a `general-purpose` subagent with the same prompt as in
   the loop, but with argument `<id> unattended` (`<id> unattended claude-plan` in
   `claude-plan` mode) and the user's decision appended verbatim
   ("The user's decision for <id>: …"). Handle its result lines like in the loop. On
   `FAILED`, stop and report.
4. Skipped items stay in `USER_PLAN.md`. Move on to the next blocked item.

A feature can make later items easier or change what they mean. After each phase-2 item, run
one more unattended loop iteration if the agent's report mentions that something else became
implementable.

## Final report

Short: the commits made (`git log --oneline <start>..HEAD`), the collected `defaults` the user
may want to tune, what couldn't be verified (UI, feel), the collected `scopes` (commits without
a scope, with the suggested scopes to add to `USER_WORKFLOW.md`), items still open (and which
of them the user chose to skip in phase 2), and the failure output if the loop ended on
`FAILED`. In `claude-plan` mode, also list the `[CONFLICT]` items built, with the part of the
design docs that is now out of date. Remind the user that nothing was pushed.
