---
name: implement-next-feature
description: Check USER_PLAN.md (the user's own feature list, never CLAUDE_PLAN.md) for open items, pick the one that is best to implement next, implement it, verify it, remove the entry from USER_PLAN.md and commit. Use when the user asks to work off their plan, implement the next feature from USER_PLAN.md, or names a user plan item to implement. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Implement the next user plan item

`USER_PLAN.md` (repo root) is the user's hand-written running list of features and ideas.
This skill works it off **one item at a time**: pick the best next item, implement it, verify
it, delete its entry and commit.

Before anything else:

- If `USER_WORKFLOW.md` is missing, the project isn't set up for this workflow: tell the user
  to run `/setup-workflow` and stop.
- Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`,
  `CLAUDE.md` and `USER_WORKFLOW.md`. Their rules apply throughout.

**Only `USER_PLAN.md`.** Never pick an item from `CLAUDE_PLAN.md`; those are only built one
by one after the user approved them through `/pitch-feature` (which reuses steps 3–5 and 7
below), or by `/implement-all-claude-features` after the user confirmed that run (which
passes `claude-plan`, see below).

**`claude-plan` mode.** Only when the argument contains `claude-plan` (passed by
`/implement-all-features` in a run the user confirmed, always together with `unattended`;
outside such a run, point the user to `/pitch-feature` instead), work off `CLAUDE_PLAN.md`
instead:

- Read `CLAUDE_PLAN.md` wherever this skill says `USER_PLAN.md`; leave `USER_PLAN.md` alone.
- `CLAUDE_PLAN.md` has no suggested-order section; its items are roughly ordered by priority
  inside each section, so use that order after the other rules in step 2.
- Items tagged **[CONFLICT]** count as needing the user (blocked in unattended mode), like
  ideas, unless the prompt includes the user's decision for them.
- Never touch the `## Dropped` section.
- Commit footer: `Refs: claude plan <id> (<title>)`.

Optional argument: an item id or title (e.g. `F2`), or a focus (e.g. `UI`, `quick`). With an
id, implement that item. With a focus, pick the best item within it. Without one, pick the
best item overall.

The argument may also contain `unattended` (used by `/implement-all-features`, which runs this
skill in a fresh subagent per item), `skip=<ids>` (comma-separated ids to leave alone) and
`claude-plan` (see above). In
**unattended mode** nobody can answer questions: never ask the user, skip items that would
need their decision instead (see step 2), and end with the lines from step 7.

## 1. Check for open points

1. Read `USER_PLAN.md`. If it holds no items, tell the user there is nothing open, then stop.
2. Read the **Design docs** named in `USER_WORKFLOW.md` (the intent the user's items build on).
3. Run `git status --short`. If the working tree has unrelated uncommitted changes, mention
   them and keep your edits clearly separate; don't touch or revert them.

## 2. Pick the item

Without an explicit id, choose using this order of preference:

1. A suggested-order section in `USER_PLAN.md`, if the user wrote one.
2. **Concrete features** before **Ideas**. Ideas are only worked on when named or with the
   user's decision, since they still need design work.
3. Items other items build on (cross-references like "needs F2").
4. Higher severity ([H] > [M] > [L]), then the smaller, lower-risk change.
5. Skip every item listed in `skip=`.
6. Never work on an item tagged **[HOLD]**, not even when named: tell the user it is on hold
   and that they can remove the tag.

Then **check the item against the current code**: it may already be (partly) there. If it is
fully done, delete the entry (see step 5), say so and pick the next one.

**Clarity check:** the user's entries can be short. Work out what they mean from the entry,
the design docs and the code. Choose sensible defaults for small details (exact numbers,
colours, wording, layout) and list them in the report so the user can tune them. If something
essential is genuinely ambiguous (several reasonable designs that behave differently), or the
item contradicts the design docs, or it is a big change (many files, an incompatible change of
a persisted or public format, renaming something `CLAUDE.md` documents as load-bearing), or it
falls under **Ask before** in `USER_WORKFLOW.md`, briefly present the planned approach and ask
the user before editing. For a clear item, state which one you picked and why in one or two
sentences, then proceed.

In unattended mode, an item that would need the user (the clarity check above, every item
from `Ideas`) is **not** worked on: note it as blocked and pick the next item. If every open
item is blocked or skipped, edit nothing and finish with `RESULT: BLOCKED` (step 7).

Exception: if the prompt names the item and includes **the user's decision** for it (an
approved approach, answers to the open questions), that counts as approval. Implement it
following that decision instead of blocking it.

## 3. Implement it

- **Notes for Claude:** in an entry are binding constraints and **Done when:** is the
  acceptance criterion; check it explicitly in step 4.
- Plan the change first: which modules, data, UI and config it touches. Prefer extending the
  existing systems `CLAUDE.md` describes (registries, data directories, state owners, UI
  patterns) over adding parallel ones.
- Keep to the conventions in `CLAUDE.md` (style, typing, naming, comments) and match the
  surrounding code.
- Respect the invariants `CLAUDE.md` documents (load-bearing names, ordering constraints,
  generated companion files, ownership rules).
- New state that has to persist must be saved the way `CLAUDE.md` describes; if a persisted
  format changes incompatibly, follow what it says about that (e.g. bump a version).
- Never edit the **Hands-off paths** from `USER_WORKFLOW.md`.
- Keep the change scoped to the item. If you notice another problem, don't fix it on the side;
  mention it in the report.

## 4. Verify

1. Run the verification from `CLAUDE.md` → `### Verification`, before and after the change
   where it compares output. There must be no new errors or warnings, and behaviour may only
   change where the feature intends it. If the feature adds state the verification should
   cover (e.g. a diagnostic dump or a test), extend it.
2. If `CLAUDE.md` has no verification section, use the project's build, lint and test commands
   you can find, and say in the report that the section is missing (`/setup-workflow check`).
3. Run the visual/manual checks the section lists if their tools are available.
4. Say exactly what you could not verify (UI, feel, persistence, release builds) and how the
   user can check it by hand.

If verification fails and you can't fix it, leave the entry in `USER_PLAN.md`, report the
failure with the output, and don't claim the item is done.

## 5. Update `USER_PLAN.md` (and `CLAUDE.md`)

- Follow `list-format.md`: delete the entry when fully done (with its mentions in a
  suggested-order section and in other entries), or rewrite it with an `_Updated:_` stamp
  under its `_Written:_` line when partly done. Leave the user's other entries exactly as they
  wrote them.
- If the feature changes something `CLAUDE.md` describes (architecture, flows, conventions,
  the Workflow section), update `CLAUDE.md` in the same change. If it settles something the
  design docs leave open, mention it in the report; don't edit the design docs unless the user
  asks.

## 6. Report back

In a few short sentences: which item you implemented and why you picked it, what you changed
(files), the defaults you chose, how you verified it and what you couldn't verify, any side
issues you noticed, the commit (with a scope suggestion if no scope fit), and how many items
are still open plus which one would be next.

## 7. Commit

After a successful, verified implementation (full or partial), commit it:

- Stage only the files of this item plus `USER_PLAN.md` (and `CLAUDE.md` or companion files if
  touched), by explicit path. Never stage unrelated changes that were already in the tree.
- Message per `USER_WORKFLOW.md` → `## Commits` (usually `feat`), scope from its scope list or
  none if none fits (see "Scope reporting" in `list-format.md`). End with the `Refs:` footer
  from `list-format.md` (`Refs: user plan <id> (<title>)`), then the attribution lines from
  the system reminder.
- If verification failed, don't commit; leave the changes in the working tree and report.

In unattended mode, end your answer with these lines:

- `BLOCKED: <id> — <one-line reason / the decision needed>` for every item you skipped as
  needing the user in this run (none if there were none).
- `DEFAULTS: <id> — <the defaults you chose>` if you chose any.
- `SCOPE: …` from `list-format.md` if the commit has no scope.
- As the very last line, exactly one of:
  - `RESULT: DONE <id> <short commit>`: entry deleted and committed.
  - `RESULT: PARTIAL <id> <short commit>`: entry rewritten to what is still open, committed.
  - `RESULT: NOTHING_LEFT`: no open items.
  - `RESULT: BLOCKED <ids>`: nothing implementable unattended; list the blocked/skipped ids.
  - `RESULT: FAILED <id>`: implementation or verification failed, nothing committed.
