---
name: fix-all-user-findings
description: Like /fix-all-findings, but only for the user's own USER_ANALYSIS.md (never CLAUDE_ANALYSIS.md): one fresh subagent per finding fixes, verifies, deletes the entry and commits, until nothing fixable is left; items needing the user's decision come last, with only the decision asked. Use when the user asks to fix all of their own findings or to work off USER_ANALYSIS.md. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Fix all user findings

This is `/fix-all-findings` restricted to `USER_ANALYSIS.md`. Follow
`${CLAUDE_PLUGIN_ROOT}/skills/fix-all-findings/SKILL.md` exactly, with `user-only` added to the argument,
followed by this skill's own argument (a maximum number of fixes and/or `skip=<ids>`, ids as
`U1.1`).

With `user-only`, that skill counts only the user's findings, passes `user-only` on to every
`fix-next-finding` subagent, and in phase 2 asks only about `U` items. `CLAUDE_ANALYSIS.md`
stays out of this run entirely; the final report may mention in one line how many items are
open there.
