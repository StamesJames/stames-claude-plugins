---
name: implement-all-features
description: Work off the concrete features in USER_PLAN.md completely (never CLAUDE_PLAN.md, never Ideas): ask all needed decisions up front, then implement one feature per fresh subagent, unattended, with review, failure handling and a closing analysis of the changes. Use when the user asks to implement all of their plan, work off USER_PLAN.md, or loop /implement-next-feature. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Implement all user plan items

Follow `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/loop.md` exactly, with these specifics:

- **Kind of work:** `features` (models `features`; fresh review and analyse-after-run settings
  for features).
- **Lists:** `## Concrete features` of `USER_PLAN.md` only. `## Ideas` is never part of a run
  (they become features through `/explore-feature`). `CLAUDE_PLAN.md` is never touched; its
  items join a run once approved into `USER_PLAN.md`.
- **Single-item skill:** `implement-next-feature`, description `Implement <id>`.
- **Triage cases** (needs the user, besides `work-item.md` B): the item contradicts the
  design docs; the item is too vague to build without guessing its behaviour (offer the
  concrete readings as options).
- **Order** for the triage: a suggested-order section if the user wrote one, then items others
  build on, then severity, then smaller first.

Arguments (all optional): a maximum number of items (`3`) and `skip=<ids>`.

The final report also lists the `DEFAULTS:` the agents chose (the user may want to tune them)
and what couldn't be verified (UI, feel).
