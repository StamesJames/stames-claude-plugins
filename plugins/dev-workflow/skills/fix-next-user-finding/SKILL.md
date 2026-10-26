---
name: fix-next-user-finding
description: Like /fix-next-finding, but only for the user's own USER_ANALYSIS.md (never CLAUDE_ANALYSIS.md): pick the user finding that is best to fix next (or a given one), fix it, verify it, delete its entry and commit. Use when the user asks to fix their own findings, the next user finding, or names a U item while asking to leave Claude's analysis alone. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Fix the next user finding

This is `/fix-next-finding` restricted to `USER_ANALYSIS.md`. Invoke the `dev-workflow:fix-next-finding`
skill with the argument `user-only` followed by this skill's own argument (if the Skill tool is
unavailable, read `${CLAUDE_PLUGIN_ROOT}/skills/fix-next-finding/SKILL.md` and follow it exactly, with
`user-only` set).

Optional argument, passed through: a user finding number (`1.1` or `U1.1`; both mean item
`1.1` in `USER_ANALYSIS.md` here, write it as `U1.1` when passing it on), a focus, and the
`unattended` / `skip=<ids>` switches.

`CLAUDE_ANALYSIS.md` stays out of this run entirely: don't pick from it, don't count its items
and don't mention them as "next". If it has open items, a single closing line saying how many
is fine.
