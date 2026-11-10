---
name: implement-all-features
description: Work off the concrete features in CLAUDE_PLAN.md completely (never ideas) - ask all needed decisions up front, then start the unattended driver (workflow-lists run go), which implements one feature per fresh headless agent with acceptance review, rework, failure handling, a closing analysis and per-agent stats. Use when the user asks to implement all of the plan, work off CLAUDE_PLAN.md, or loop /implement-next-feature.
---

# Implement all planned features

Follow `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/loop.md` exactly, with these specifics:

- **Kind of work:** `features` (`workflow-lists run … features`; the `features` model,
  review and analyse-after-run settings for features).
- **Lists:** `CLAUDE_PLAN.md` only. Ideas (`USER_IDEAS.md`, `CLAUDE_IDEAS.md`) are never part
  of a run; they become features through `/plan-features`.
- **Triage cases** (needs the user, besides `work-item.md` B): the item contradicts the
  design docs and its **Decisions:** don't say to go ahead; the item is too vague to build
  without guessing its behaviour (offer the concrete readings as options).
- **Triage depth:** plan entries were already decided with the user by `/plan-features`, so
  `workflow-lists triage` marks most `READY` from the entry alone, and the triage agent runs
  only when it marks some `CHECK` (`[CONFLICT]`, an entry written or edited by hand, cited
  files changed since it was written). For those, look at the code as far as the reason
  needs. The implementing agent checks the code again anyway (`work-item.md` B), and blocks
  an item that turns out to need the user.

Arguments (all optional): a maximum number of items (`3`) and `skip=<ids>`.

If `CLAUDE_PLAN.md` holds no open features but `USER_IDEAS.md` has ideas, suggest
`/plan-features` instead of stopping silently.
