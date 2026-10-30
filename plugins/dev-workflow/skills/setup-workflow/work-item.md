# Working off one item: shared steps

Read by the single-item skills (`fix-next-finding`, `implement-next-feature`, and
`pitch-feature` after approval). They decide *which* item and add their specifics (the list
file, the commit type, the result word); the steps below are the same for all of them.
Section letters are referenced from those skills; keep them stable.

## A. Modes and arguments

- **`unattended`** (set by the loop skills, which run one fresh subagent per item): nobody
  can answer questions. Never ask; an item that needs the user is **blocked** instead (B).
  Keep your final answer short: at most three plain lines, then the lines from H. The whole
  answer lands in the orchestrator's context.
- **An id** (always given by the loops): work on that item only; don't read or weigh the
  other entries of the list beyond what B needs.
- **`skip=<ids>`**: ids to leave alone when picking yourself.
- **The user's decision** for the item may be in the entry (`**Decisions:**`) or appended to
  the prompt. Either counts as approval for what it covers.

Read only what the item needs: its own entry, the files it names, the code around them, and
of the design docs only the sections the entry's **Design docs:** line names (if it names
none and the item touches design intent, open the docs `USER_WORKFLOW.md` lists and read only
the relevant sections). Read `**Keep open for:**` ideas by id, not the whole Ideas section.

Run `git status --short` first. If the working tree has unrelated changes, mention them and
keep your edits clearly separate; don't touch or revert them. Note `git rev-parse --short
HEAD` as the starting commit.

## B. Check the item and whether it needs the user

1. **Check it against the current code.** Line numbers may have moved and the code may have
   changed since the stamp. Already done: delete the entry (F, then G with type `chore` and
   message `chore: remove done item <id>`), say so, and in unattended mode finish with
   `RESULT: DONE <id> <commit>`. Partly done: narrow the work to what is still open.
2. **Needs the user** when, and no recorded decision covers it:
   - it falls under **Ask before** in `USER_WORKFLOW.md`;
   - it is a big change: many files, an incompatible change of a persisted or public format,
     renaming something `CLAUDE.md` documents as load-bearing;
   - something essential is genuinely ambiguous (several reasonable designs that behave
     differently);
   - plus the skill's own cases (e.g. "verify" findings, `[CONFLICT]` items).

   Small details (exact numbers, colours, wording, layout) are not ambiguous: choose sensible
   defaults and list them in the report (`DEFAULTS:` line in unattended mode).
3. Attended: briefly present the planned approach and ask before editing. Unattended: don't
   work on it. Report it with a `BLOCKED:` line; with an id given, finish with
   `RESULT: BLOCKED <id>`. If you notice this only after editing, undo your own edits first
   (the tree was clean when you started, apart from changes you were told about).

## C. Build it

- **Notes for Claude:** and **Decisions:** are binding; **Done when:** is the acceptance
  criterion, checked explicitly in E.
- Plan the change first: which modules, data, UI and config it touches. Prefer extending the
  existing systems `CLAUDE.md` describes (registries, data directories, state owners, UI
  patterns) over adding parallel ones, and reuse existing helpers over writing new ones
  (search for them).
- Follow `USER_WORKFLOW.md` → `## Code standards` and the conventions in `CLAUDE.md` (style,
  typing, naming, comments), and match the surrounding code.
- Respect the invariants `CLAUDE.md` documents (load-bearing names, ordering constraints,
  generated companion files, ownership rules). New state that has to persist is saved the way
  `CLAUDE.md` describes; if a persisted format changes incompatibly, follow what it says (e.g.
  bump a version).
- **Keep open for:** don't block what it names, don't build for it (see `list-format.md`).
- **Tests:** if the project has a test suite, new behaviour and bug fixes get tests (a bug
  fix: a test that fails without the fix), unless `## Code standards` says otherwise.
- Never edit the **Hands-off paths** from `USER_WORKFLOW.md`.
- Keep the change scoped to the item. If you notice another problem, don't fix it on the side;
  mention it in the report (it can go into the next `/analyse-code` pass).

## D. Self-review

Before verifying, read your own `git diff` once as a reviewer would, and fix what you find:

- anything outside the item's scope, debug leftovers, commented-out or dead code;
- duplication of code that already exists in the project, or of your own new code;
- names, structure and error handling that don't match `## Code standards` and the
  surrounding code;
- abstractions, options or hooks nothing uses yet;
- missing tests (C), stale comments, a `CLAUDE.md` that no longer describes the code.

Keep it short: one pass over the diff, not a second implementation round.

## E. Verify

1. Run the verification from `CLAUDE.md` → `### Verification`. Where it compares output
   before and after, run it before editing too if the change may alter behaviour. It must pass
   with no new errors or warnings, and behaviour may only change where the item intends it.
2. If `CLAUDE.md` has no verification section, use the project's build, lint and test commands
   you can find, and say in the report that the section is missing (`/setup-workflow check`).
3. Run the visual/manual checks the section lists if their tools are available.
4. Check **Done when:** explicitly. Say exactly what you could not verify (UI, feel,
   persistence, release builds) and how the user can check it by hand.

If verification fails and you can't fix it, leave the entry as it is, don't commit, leave the
changes in the working tree, report the failure with the output, and don't claim the item is
done.

## F. Update the list (and `CLAUDE.md`)

- Follow `list-format.md`: delete the entry when fully done (with its mentions in a
  suggested-order section, in other entries and in `**Keep open for:**` lines), or rewrite it
  with an `_Updated:_` stamp when partly done. Leave other entries exactly as they are. Don't
  add a line to a **Passes** list.
- If the deleted entry names `workflow/failed/...` branches, delete them (`git branch -D`).
- If the change alters something `CLAUDE.md` describes (architecture, flows, conventions,
  commands, the Workflow section), update `CLAUDE.md` in the same change. If it settles
  something the design docs leave open, mention it in the report; don't edit the design docs
  unless the user asks.

## G. Commit

After a successful, verified change (full or partial):

- Stage only the files of this item plus the list file you edited (and `CLAUDE.md` or
  companion files if touched), by explicit path. Never stage unrelated changes.
- Message per `USER_WORKFLOW.md` → `## Commits`, with the type the skill names, scope from its
  scope list or none if none fits (see "Scope reporting" in `list-format.md`). End with the
  `Refs:` footer from `list-format.md`, then the attribution lines from the system reminder.

## H. Report

**Attended:** in a few short sentences: which item and why, what you changed (files), the
defaults you chose, how you verified it and what you couldn't, side issues you noticed, the
commit (with a scope suggestion if no scope fit), and how many items are still open plus which
one would be next.

**Unattended:** at most three plain lines (what changed, what couldn't be verified, side
issues), then these lines:

- `BLOCKED: <id> — <the decision needed>` for every item you skipped as needing the user.
- `DEFAULTS: <id> — <the defaults you chose>` if you chose any.
- `SCOPE: …` from `list-format.md` if the commit has no scope.
- As the very last line, exactly one of:
  - `RESULT: DONE <id> <short commit>`: entry deleted and committed.
  - `RESULT: PARTIAL <id> <short commit>`: entry rewritten to what is still open, committed.
  - `RESULT: NOTHING_LEFT`: no open items (only when picking yourself).
  - `RESULT: BLOCKED <ids>`: nothing workable unattended; the blocked/skipped ids.
  - `RESULT: FAILED <id> — <one-line reason>`: build or verification failed, nothing
    committed, changes left in the working tree.
