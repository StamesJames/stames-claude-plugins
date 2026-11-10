---
name: pitch-feature
description: Pick the single idea from CLAUDE_IDEAS.md (Claude's own suggestions) that would be best right now, explain what it is and how it benefits the project, and on the user's answer add it to USER_IDEAS.md, plan it right away with /plan-features, or drop it. Never changes project code. Use when the user asks what Claude would build next, or for a feature pitch or recommendation from Claude's ideas.
---

# Pitch one of Claude's ideas

`CLAUDE_IDEAS.md` (repo root) holds Claude's own suggestions, written by `/suggest-features`.
Nothing in it is planned or built without the user's approval. This skill pitches **one**
idea; the user then adds it to their ideas in `USER_IDEAS.md` (so `/plan-features` plans it),
has it planned right away, or drops it. Never loop, never run this skill unattended or in a
subagent, and never change project code.

Optional argument: an idea id (`I3` or `claude:I3`) to pitch that one, or a focus (`quick`,
`onboarding`) to pick within it.

## 1. Check for open ideas

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. `CLAUDE.md` is already in your context (read the file only if it isn't). Read part 1 of
   `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md` (`sed '/^# Part 2/,$d'
   <file>`), `USER_WORKFLOW.md` and `CLAUDE_IDEAS.md`. If it holds no open ideas, say so,
   suggest `/suggest-features`, and stop.
3. Read the design-doc sections the candidates name,
   `workflow-lists summary --ideas USER_IDEAS.md CLAUDE_PLAN.md` (`show` the entries that may
   overlap a candidate) and
   `git log --oneline -15`, so you know the intent and what the user is working on right now.

## 2. Pick the best idea for *now*

Not simply the highest severity. Weigh:

- Value for the project at its current stage: what its users miss most right now.
- Fit with recent work, `CLAUDE_PLAN.md` and `USER_IDEAS.md` (builds on what was just made,
  unblocks or complements the user's own plans; doesn't duplicate or collide with them).
- Effort and risk: a smaller idea with nearly the same value wins.
- `[CONFLICT]` ideas only if they are clearly the best; the pitch must then say so up front.
- Never ideas tagged **[HOLD]**, not even when named, unless the user lifts the hold.

**Check the idea against the current code and the plan** before pitching: if it is already
done or planned, `workflow-lists delete claude:<id>`, say so and pick another one. If the code moved on, adjust
the idea for the pitch.

## 3. Pitch it

Tell the user, short and in plain words:

- **What** it is, as the user of the product experiences it.
- **Why now**: how it benefits the project and why this one beats the other open ideas.
- **How**: the rough approach, the size, and the risks (persisted formats, behaviour changes,
  anything under **Ask before**).
- **Conflict**: for a `[CONFLICT]` idea, what in the design docs it contradicts and why it's
  worth it anyway.

Then ask with `AskUserQuestion`:

- `Plan it now` (recommended for a small, clear idea) — move it to `USER_IDEAS.md` (step 4),
  then plan it (step 5).
- `Add to my ideas` (recommended otherwise) — move it to `USER_IDEAS.md` (step 4); the next
  `/plan-features` picks it up.
- `Pitch another` — pitch the next best idea instead (once; if that is turned down too, stop).
- `Drop it` — `workflow-lists delete claude:<id> --dropped "<the user's reason>"` (it adds the
  `## Dropped` line). If the user gave no reason, ask for one in a single short question
  (they may answer "none"). Commit
  `CLAUDE_IDEAS.md` (type `chore`, e.g. `chore: drop idea <name>`), then stop.

"Not now" or any other answer (the user's own text) keeps the idea and stops, or, if it
changes the idea, adapt the pitch and ask again.

## 4. Move it to `USER_IDEAS.md`

`workflow-lists move claude:<id> USER_IDEAS.md --note "<the user's remarks from the pitch>"`
(leave out `--note` if there were none). It gives the idea a new id and stamp, keeps its text,
deletes it from `CLAUDE_IDEAS.md` and points `**Keep open for:**` mentions to the new id. Fix
every `error` it prints. Commit the files you edited (type `chore`, e.g.
`chore: add idea pause menu sound (I9)`, no `Refs:` footer).

## 5. Plan it now (only after `Plan it now`)

Invoke the `dev-workflow:plan-features` skill with the new id (if the Skill tool is
unavailable, read `${CLAUDE_PLUGIN_ROOT}/skills/plan-features/SKILL.md` and follow it). The
questions it asks build on what the user already said in the pitch.

## 6. Report back

In a few short sentences: the new `USER_IDEAS.md` id (or the features planned from it, with
the next step `/implement-next-feature <id>`), and how many ideas are left in
`CLAUDE_IDEAS.md`. Don't offer to pitch the next one right away.
