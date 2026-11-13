## Workflow

This project uses the `dev-workflow` plugin's skills (`/analyse-code`, `/fix-next-finding`,
`/fix-all-findings`, `/suggest-features`, `/pitch-feature`, `/brainstorm-feature`,
`/plan-features`, `/implement-next-feature`, `/implement-all-features`, `/review-change`,
`/setup-workflow`). The running lists `CLAUDE_ANALYSIS.md`, `USER_ANALYSIS.md`,
`USER_IDEAS.md`, `CLAUDE_IDEAS.md` and `CLAUDE_PLAN.md` sit at the repo root; their rules are in `list-format.md` of the
`dev-workflow` plugin's `setup-workflow` skill. The user's preferences for them (design docs,
idea sources, code standards, hands-off paths, what to ask before, run settings, commit rules
and scopes) are in `USER_WORKFLOW.md`. This section holds the facts; keep it current when the
code changes. The lists may be newer than this file: if they disagree, check the code. When a
finding or plan item is resolved during normal work, delete its entry in the same change.

### Code scope

- Project code: <dirs/globs>, plus <config/data files a finding may depend on>.
- Generated / not project code: <dirs>. (Hands-off paths are in `USER_WORKFLOW.md`.)
- List paths are relative to `<code root>`.

### Verification

How to check a change, in tiers. An agent runs **Quick**, the **Targeted** checks of the areas
its change touches, and **Full** only in the cases named there or when the item's **Verify:**
asks for it. A section without tier labels counts as Quick: everything, every change.

**Quick** (every change):

1. `<command>`: <what must hold, e.g. builds with no new errors or warnings>.
2. `<command>`: <what to compare before/after, what must not appear>.

**Targeted** (by area; drop this block if every check is quick):

- `<dir or glob>` → `<command>`: <what it checks>.

**Full** (slow; only when <the cases, e.g. a save-format or engine change>; drop this block if
there is none):

1. `<command>`: <what must hold>.
2. <optional visual/manual check and the tool for it, if available>.

<What these cannot exercise, so the report names it.>
