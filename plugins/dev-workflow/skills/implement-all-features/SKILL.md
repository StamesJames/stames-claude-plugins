---
name: implement-all-features
description: Work off the concrete features in CLAUDE_PLAN.md completely (never ideas): ask all needed decisions up front, then implement one feature per fresh subagent, unattended, with review, failure handling and a closing analysis of the changes. Use when the user asks to implement all of the plan, work off CLAUDE_PLAN.md, or loop /implement-next-feature. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Implement all planned features

Follow `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/loop.md` exactly, with these specifics:

- **Kind of work:** `features` (models `features`; fresh review and analyse-after-run settings
  for features).
- **Lists:** `CLAUDE_PLAN.md` only. Ideas (`USER_IDEAS.md`, `CLAUDE_IDEAS.md`) are never part
  of a run; they become features through `/plan-features`.
- **Single-item skill:** `implement-next-feature`, description `Implement <id>`.
- **Triage cases** (needs the user, besides `work-item.md` B): the item contradicts the
  design docs and its **Decisions:** don't say to go ahead; the item is too vague to build
  without guessing its behaviour (offer the concrete readings as options).
- **Order** for the triage: the suggested-order section if there is one, then items others
  build on, then severity, then smaller first.

Arguments (all optional): a maximum number of items (`3`) and `skip=<ids>`.

If `CLAUDE_PLAN.md` holds no open features but `USER_IDEAS.md` has ideas, suggest
`/plan-features` instead of stopping silently.
