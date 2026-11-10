---
name: brainstorm-feature
description: Talk through a rough feature idea of the user's own (given as the argument) in a loose conversation - what they want, why, alternatives, what the code allows - and once it is clear, plan it straight into CLAUDE_PLAN.md with /plan-features, without going through USER_IDEAS.md (or park it there). Never changes project code. Use when the user wants to brainstorm, discuss, think through or talk about an idea of their own before it is planned. Not for Claude's own ideas (/suggest-features, /pitch-feature).
---

# Brainstorm one idea, then plan it

The user brings a rough idea and wants to talk it over before anything is written down. This
skill has two phases: first a loose conversation until the user knows what they want, then the
`plan-features` flow on the result, which asks the questions an implementation needs and
writes the features into `CLAUDE_PLAN.md`. The idea skips `USER_IDEAS.md` unless the user parks
it. Never change project code, and never run this skill unattended or in a subagent.

Argument: the idea, in the user's words (`maybe some kind of undo`). Without one, ask what they
have in mind in one plain sentence and wait.

## 1. Context

1. If `USER_WORKFLOW.md` is missing, tell the user to run `/setup-workflow` and stop.
2. Use `CLAUDE.md` (already in your context; read the file only if it isn't). Read the design
   docs named in `USER_WORKFLOW.md` (the parts the idea touches), `USER_IDEAS.md`,
   `CLAUDE_IDEAS.md` and `CLAUDE_PLAN.md`. Skim the code the idea would touch (read-only),
   enough to talk about it concretely.
3. If the idea is already done, or an entry in a list covers it, say so up front (with the id)
   and ask whether to build on that entry, still talk it through, or stop. Naming a
   `CLAUDE_IDEAS.md` idea as the starting point counts as approving it (step 3 hands on its
   `claude:I<n>` id).

## 2. Talk it through

A conversation in plain text, over as many turns as the user wants; no `AskUserQuestion` forms
and no spec yet. Your job is to help the user find out what they want, not to collect answers:

- Say back how you understand the idea and what problem it solves for the user of the product.
- Ask one or two open questions per turn, about the goal and the experience, not the
  implementation: who uses it, when, what it should feel like, what it must not do.
- Bring in what you know: what the code already has or makes easy or hard, how the design
  docs see it, related entries in the lists, how other products solve it. Offer two or three
  directions when the idea is still wide, with your own view on which fits best.
- Push back where something clashes with the design docs or seems not worth its cost; the
  user decides.
- Let the idea change, grow or shrink. If it splits into several ideas, say so and agree which
  one to carry on with; the others can be parked (step 4).

Leave out what `plan-features` will settle later (edge cases, exact scope, done-criteria,
sizes) unless the user brings it up.

When the idea seems settled (the user says so, or the last turns only confirm it), sum it up
and ask with `AskUserQuestion`:

- `Plan it now` (recommended) — step 3.
- `Keep talking` — back to the conversation.
- `Park as idea` — step 4.
- `Drop it` — nothing is written; stop.

The summary: the idea in two to five plain lines, as the user of the product would see it,
followed by **Agreed:** (what the user decided, in their words where they gave them) and
**Open:** (what was left undecided).

## 3. Plan it

Invoke the `dev-workflow:plan-features` skill with the summary as its free-text wish (or with
the `claude:I<n>` id from step 1); if the Skill tool is unavailable, read
`${CLAUDE_PLUGIN_ROOT}/skills/plan-features/SKILL.md` and follow it. It plans in depth. The
**Agreed:** points count as answered: don't ask them again, carry them into the entries'
**Decisions:**, and ask only what is still open or what an implementation needs. Its steps
write `CLAUDE_PLAN.md`, commit and report; if the user says `Not yet` there, it keeps the wish
as an idea in `USER_IDEAS.md`.

## 4. Park it

`workflow-lists add USER_IDEAS.md --title "<title>"` with the body on stdin: the summary as
its text, the **Agreed:** points as `**Decisions:** … (<date>)` and the **Open:** points as
**Notes for Claude:**, so a later `/plan-features` starts from them. Fix every `error` it
prints. Commit only `USER_IDEAS.md`, by explicit path (type `chore`, e.g.
`chore: add idea undo last move (I7)`, no `Refs:` footer; message per `USER_WORKFLOW.md` →
`## Commits`; end with the attribution lines from the system reminder). Report the new id and
that `/plan-features <id>` plans it later.
