# Working off one item: shared steps

Read by the single-item skills (`fix-next-finding`, `implement-next-feature`) and, for
sections C to E and G, by the rework agent of a run. The skills decide *which* item and add
their specifics (the list file, the commit type); the steps below are the same for all of
them. Section letters are referenced from those skills; keep them stable.

## A. Modes and arguments

- **`unattended`** (set by the runs, which start one fresh headless agent per item): nobody
  can answer questions. Never ask; an item that needs the user is **blocked** instead (B).
  Keep your final answer short: at most three plain lines, then the lines from H. The whole
  answer is parsed by `workflow-lists run`, which only keeps those lines.
- **An id** (always given by the runs): work on that item only; don't read or weigh the
  other entries of the list beyond what B needs.
- **`skip=<ids>`**: ids to leave alone when picking yourself.
- **The user's decision** for the item may be in the entry (`**Decisions:**`) or appended to
  the prompt. Either counts as approval for what it covers.

**Context:** `CLAUDE.md` is already in your context (read the file only if it isn't). Once
you know the id, `workflow-lists brief <id>` prints everything else in one go: the entry,
the entries its **Keep open for:** names, the item rules, the sections of `USER_WORKFLOW.md`
that apply (design docs, code standards, hands-off paths, ask-before rules, commit rules) and
`CLAUDE.md`'s `### Code scope` and `### Verification`, plus the current `HEAD` and whether
the tree is clean. Don't read those files whole. Then read only what the item needs: the
files it names, the code around them, and of the design docs only the sections the entry's
**Design docs:** line names (if it names none and the item touches design intent, open the
docs `USER_WORKFLOW.md` lists and read only the relevant sections).

If the brief shows uncommitted changes, they aren't yours: mention them, keep your edits
clearly separate, don't touch or revert them. Note the `HEAD` it printed as the starting
commit.

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
   (the tree was clean when you started, apart from changes the brief told you about).

## C. Build it

- **Done when:** is the acceptance criterion and **Verify:** how it is checked; both are
  checked explicitly in E. If the entry has no **Verify:** line, decide before building how
  you will show that Done when holds (a test, a command, an observable behaviour) and say so
  in the report.
- Plan the change first: which modules, data, UI and config it touches. Prefer extending the
  existing systems `CLAUDE.md` describes (registries, data directories, state owners, UI
  patterns) over adding parallel ones, and reuse existing helpers over writing new ones
  (search for them).
- Follow `## Code standards` and the conventions in `CLAUDE.md` (style, typing, naming,
  comments), and match the surrounding code.
- Respect the invariants `CLAUDE.md` documents (load-bearing names, ordering constraints,
  generated companion files, ownership rules). New state that has to persist is saved the way
  `CLAUDE.md` describes; if a persisted format changes incompatibly, follow what it says (e.g.
  bump a version).
- **Tests:** if the project has a test suite, new behaviour and bug fixes get tests (a bug
  fix: a test that fails without the fix), unless `## Code standards` says otherwise. A run
  reports an item whose commits touched no test file.
- Keep the change scoped to the item. If you notice another problem, don't fix it on the side;
  report it as a `NOTE:` line (it can go into the next `/analyse-code` pass).

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
3. Run the item's **Verify:** steps and check **Done when:** explicitly. Run the visual or
   manual checks the verification section lists if their tools are available. Say exactly
   what you could not verify (UI, feel, persistence, release builds) and how the user can
   check it by hand (`UNVERIFIED:` line).
4. **Budget:** if verification fails, fix and rerun at most twice. If it still fails, stop:
   leave the entry as it is, don't commit, leave the changes in the working tree, report the
   failure with the output (`RESULT: FAILED`), and don't claim the item is done. A run saves
   the attempt on a branch and may retry with a stronger model.

## F. Update the list (and `CLAUDE.md`)

- Fully done: `workflow-lists delete <id>`. It deletes the entry with its mentions and failed
  branches, checks the lists and prints the `Refs:` line for G; fix by hand the mentions it
  prints as `check this mention yourself` and any `error`.
- Partly done: rewrite the entry to what is still open (Read the file from its line, Edit),
  then `workflow-lists set <id> updated`.
- Leave other entries exactly as they are. Don't add a line to a **Passes** list.
- If the change alters something `CLAUDE.md` describes (architecture, flows, conventions,
  commands, the Workflow section), update `CLAUDE.md` in the same change. If it settles
  something the design docs leave open, mention it in the report; don't edit the design docs
  unless the user asks.

## G. Commit

After a successful, verified change (full or partial):

- Stage only the files of this item plus the list file you edited (and `CLAUDE.md` or
  companion files if touched), by explicit path. Never stage unrelated changes.
- Message per `## Commits` from the brief, with the type the skill names, scope from its
  scope list or none if none fits. End with the `Refs:` footer `workflow-lists delete`
  printed (`, partial` inside the parentheses for a partial fix), then the attribution lines
  from the system reminder.

## H. Report

**Attended:** in a few short sentences: which item and why, what you changed (files), the
defaults you chose, how you verified it and what you couldn't, side issues you noticed, the
commit (with a scope suggestion if no scope fit), and how many items are still open plus which
one would be next.

**Unattended:** at most three plain lines (what changed), then these lines, each on its own
line:

- `BLOCKED: <id> — <the decision needed>` for every item you skipped as needing the user.
- `DEFAULTS: <id> — <the defaults you chose>` if you chose any.
- `UNVERIFIED: <id> — <what you couldn't verify and how the user can check it>` if anything.
- `NOTE: <id> — <a side issue you noticed, or what you had to decide about verification>`,
  one per note; they reach the run's report.
- `SCOPE: <short commit> none — suggest <scope> (<what it would cover>)` if the commit has no
  scope.
- As the very last line, exactly one of:
  - `RESULT: DONE <id> <short commit>`: entry deleted and committed.
  - `RESULT: PARTIAL <id> <short commit>`: entry rewritten to what is still open, committed.
  - `RESULT: NOTHING_LEFT`: no open items when picking yourself, or the given id's entry no
    longer exists.
  - `RESULT: BLOCKED <ids>`: nothing workable unattended; the blocked/skipped ids.
  - `RESULT: FAILED <id> — <one-line reason>`: build or verification failed, nothing
    committed, changes left in the working tree.
