---
name: implement-all-claude-features
description: Like /implement-all-features, but for Claude's own CLAUDE_PLAN.md (never USER_PLAN.md), and only after the user explicitly confirms the run: one fresh subagent per item implements, verifies, deletes the entry and commits, until nothing implementable is left; ideas and [CONFLICT] items come last, with only the decision asked. Use when the user asks to implement all of Claude's plan, work off CLAUDE_PLAN.md completely, or build all pitched/planned Claude features at once (for a single item use /pitch-feature). Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Implement all Claude plan items

This is `/implement-all-features` for `CLAUDE_PLAN.md`. Follow
`${CLAUDE_PLUGIN_ROOT}/skills/implement-all-features/SKILL.md` exactly, with `claude-plan`
added to the argument, followed by this skill's own argument (a maximum number of items
and/or `skip=<ids>`, ids as `F3`).

With `claude-plan`, that skill first shows the open items and asks the user to confirm the
run (nothing is built without that confirmation), passes `claude-plan` on to every
`implement-next-feature` subagent, and in phase 2 asks about ideas and `[CONFLICT]` items.
`USER_PLAN.md` stays out of this run entirely; the final report may mention in one line how
many items are open there.
