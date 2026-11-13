# Rules for working on one list item

Printed by `workflow-lists brief <id>` for the agent that builds, fixes, reworks or reviews an
item. The full list rules are in `list-format.md` next to this file; an item agent needs only
these.

- The entry is the contract. **Notes for Claude:** and **Decisions:** are binding;
  **Done when:** is the acceptance criterion and **Verify:** says how to check it. A short
  entry (a symptom, no paths) is not a reason to skip it: find the cause in the code.
- Verification by tier (`work-item.md` E): **Quick** always, the **Targeted** checks of the
  areas the diff touches, **Full** only in its named cases or when **Verify:** asks for it;
  widen when unsure; an untiered `### Verification` is all Quick. Report what you ran as
  `CHECKS: <id> — <tiers and checks> (<why>)` before the `RESULT:` line.
- **Keep open for:** names ideas or needs the change must not block: avoid choices that would
  make them expensive later. It never means building for them: no speculative abstractions,
  options or hooks.
- Never work on an item tagged **[HOLD]**, not even when named.
- Edit the lists only through `workflow-lists`: `delete <id>` when the item is done (it
  removes the entry with its mentions and failed branches, and prints the `Refs:` line for
  the commit), `set <id> updated` after rewriting a partly done entry to what is still open.
  Leave every other entry exactly as it is; don't add a line to a **Passes** list.
- Never edit the **Hands-off paths**. Keep the change scoped to the item; another problem
  you notice goes into a `NOTE:` line of the report, not into the change.
- Commit message per `## Commits` below; the footer is the `Refs:` line `workflow-lists
  delete` printed (`Refs: plan F2 (<title>)`, `Refs: analysis 1.4 (<title>)`, `Refs: user
  analysis 1.1 (<title>)`; add `, partial` for a partial fix, `, rework` for a rework
  commit, `, review` for a review follow-up), then the attribution lines from the system
  reminder. If no scope from `### Scopes` fits, commit without one and report it
  (`SCOPE: <short commit> none — suggest <scope> (<what it would cover>)`).
