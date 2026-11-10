# Workflow lists: shared format and rules

Read by every workflow skill. A project uses this workflow when it has a `USER_WORKFLOW.md` at
its root. Two more shared files sit next to this one: `work-item.md` (how one item is built,
verified and committed) and `loop.md` (how a run works off a whole list).

This file has two parts. **Part 1** (up to `# Part 2`) holds the rules every skill that
touches a list needs. **Part 2** holds the files and their entry formats, needed only by
skills that write or reorder entries (analysis, planning, ideas, setup). Skills that work off
a single item read only part 1: `sed '/^# Part 2/,$d' <this file>`.

`CLAUDE.md` holds the **facts** the skills need (`## Workflow` → `### Code scope`,
`### Verification`, plus the architecture, conventions and invariants it documents anyway);
Claude keeps it current. `USER_WORKFLOW.md` holds the user's **decisions and preferences**
(design docs, idea sources, analysis focus, code standards, hands-off paths, what to ask
before, run settings, commit rules); Claude edits it only when the user asks. If the two
disagree, `USER_WORKFLOW.md` wins on preferences and the code wins on facts. The lists may be
newer than `CLAUDE.md`; if they disagree, check the code.

## The `workflow-lists` command

The plugin puts `workflow-lists` on the Bash tool's `PATH`. Run it from the repo root; it
prints a few lines, so use it instead of reading whole files:

- `workflow-lists check [<list file>...]`: checks the mechanical rules below (sections,
  `_Next id:_` lines, ids, `_No open items._`, severities, stamps, `**Model:**` lines,
  required fields of Claude's entries, `**Keep open for:**` and suggested-order references,
  `workflow/failed/...` branches, outdated headers; with no file also `CLAUDE.md`'s
  `## Workflow` section and `USER_WORKFLOW.md` against its template). It prints `OK: …`, or
  one `error` / `warn` line per problem and exits 1 if there are errors. Run it after editing a
  list and before committing, and fix every `error` in what you changed. `warn` lines in the
  user's own entries are the user's to fix: mention them, don't edit those entries.
- `workflow-lists summary [--ideas] [<list file>...]`: one line per open item (id as used
  outside the files, tags, title, line number, size, model, decisions, failed branch), held
  items included and tagged, ideas only with `--ideas`, and a `TOTAL:` line.
- `workflow-lists show <id>...`: prints just those entries (ids as used outside the files,
  `1.4`, `U1.1`, `F2`, `I3`, `claude:I2`), each under a `==> <file>:<line>` line. Use it
  instead of reading a whole list for one entry; to edit the entry, read the file from that
  line (Read with `offset`).
- `workflow-lists section <file> <heading>...`: prints those `## ` sections of a file
  (heading prefix, any case), e.g. `workflow-lists section USER_WORKFLOW.md "Code standards"`.
- `workflow-lists stamp`: the stamp value, `YYYY-MM-DD HH:MM TZ (`commit`)`.

A hook also runs the check after every edit to a list file and reports errors in that file
(including references into it that the edit left dangling). If you are mid-change (an entry
deleted, its mentions not yet), finish the change; otherwise fix what it reports.

If the command is missing (`command not found`), do these steps by hand from the rules below
(read the entry or section directly, e.g. with `grep -n` and `sed -n`).

## Rules for all lists

- **Running lists of open items only.** Never start a list from scratch; update it.
- **Done means deleted.** When an item is fixed or implemented, delete the whole entry, its
  mentions in a suggested-order section and its cross-references in other entries (including
  `**Keep open for:**` lines), in the same change. Don't strike through, don't mark as done.
  Git is the archive: the commit's `Refs:` footer (below) finds it again.
- **Partly done:** rewrite the entry to what is still open and add
  `_Updated: <date time TZ> (<commit>)_` under its `_Found:_` / `_Written:_` line.
- **Ids are never reused or renumbered;** gaps are expected. Every section holding items starts
  with a `_Next id: <id>_` line: take new ids from it and count it up in the same edit. Don't
  search the git history for ids.
- A section left without items gets back its `_No open items._` line.
- **Stamps:** `_Found: YYYY-MM-DD HH:MM TZ (`<short commit>`)_` (analysis) or `_Written: …_`
  (plan and ideas). Take the value from `workflow-lists stamp` (by hand: `date '+%Y-%m-%d %H:%M %Z'`
  and `git rev-parse --short HEAD`, `no commit` if the repo has none yet); never guess the
  time. Keep the stamps of other entries as they are.
- **[HOLD]** after the severity: the item is not touched (fixed, implemented, planned, pitched)
  until the user removes the tag, even when named explicitly. Held items are not "blocked" and
  don't count as open for `NOTHING_LEFT`. A pass may still delete a held item that turns out
  done.
- **The user's entries** (`USER_*`): **Notes for Claude:** are binding constraints, **Done
  when:** is the acceptance criterion. Entries are often short (a symptom, no paths, no fix):
  that is not a reason to skip them. Edit only the entry being resolved and its mentions;
  leave everything else exactly as the user wrote it. The only lines Claude adds to a user
  entry it isn't resolving are the run lines below (`**Decisions:**`, `_Failed:_`).
- **Suggested order** sections (section 4 of the analysis files, `## Suggested order` in the
  plan) name items in bold (`**1.1**`, `**U1.1**`, `**F2**`); only bold ids count as
  references, so version numbers in the prose are safe.

## Optional entry lines (all lists)

These may appear in any entry, right before the closing text or at its end:

- `**Decisions:** <the user's answers> (<date>)`: decisions the user made for this item,
  recorded by a run's decision step (`loop.md`) or by `/plan-features`. Binding like
  **Notes for Claude:**; an item with decisions covering its open questions no longer needs
  the user.
- `**Keep open for:** <ideas or free-text needs>`: ideas (`I2`, `claude:I4`) or future needs
  the implementation must **not block**: avoid choices that would make them expensive later
  (a hard-coded single instance where a list is planned, a format that can't be extended). It
  does **not** mean building for them: no speculative abstractions, hooks, options or config
  for features that don't exist yet.
- `**Model:** <model>[, reasoning <level>]`: which model (and reasoning effort) a run uses
  for this item, overriding the defaults in `USER_WORKFLOW.md` → `## Runs`. Written only when
  it differs from that default (rubric in part 2). Models: `haiku`, `sonnet`, `opus` (or any
  model the Agent tool accepts); levels: `low`, `medium`, `high`, `xhigh`.
- `_Failed: <date time TZ> (`<wip branch>`, <model>): <one-line reason>_` under the
  `_Found:_` / `_Written:_` line: a run tried the item and failed; the attempt is saved on the
  named branch. If the attempt left no changes to save, write `(no changes, <model>)` instead
  of the branch. Written only by `loop.md`. Such an item needs the user's decision before the
  next run touches it; the decision step turns the stamp into a `**Decisions:**` line that
  still names the branch (if any).

When an entry is deleted as done, also delete the `workflow/failed/...` branches it names
(`git branch -D`), in the same run.

## Ids outside the files

In arguments, `skip=` lists, reports, `RESULT:` lines, branch names and commit footers:

- `CLAUDE_ANALYSIS.md` items: bare id (`1.4`); in commits `analysis 1.4`; WIP branch
  `workflow/failed/analysis-1.4`.
- `USER_ANALYSIS.md` items: `U` prefix (`U1.1` = item `1.1` there); in commits
  `user analysis 1.1`; WIP branch `workflow/failed/user-analysis-1.1`.
- `CLAUDE_PLAN.md` items: bare id (`F2`); in commits `plan F2`; WIP branch
  `workflow/failed/plan-F2`.
- `USER_IDEAS.md` items: bare id (`I1`; a heading without one goes by a short slug of its
  title).
- `CLAUDE_IDEAS.md` items: `claude:I2`.

## Commit footer

Every commit that resolves (or partly resolves) a list item ends its message, before the
attribution lines, with one `Refs:` line per item, carrying the item's title so
`git log --grep 'Refs:' --oneline` reads as a log of finished work:

```
Refs: analysis 1.4 (money label not updated on load)
Refs: plan F2 (pause menu sound)
```

Add `, partial` inside the parentheses for a partial fix and `, review` for a follow-up
cleanup from `/review-change`. Everything else about the commit message (types, scopes,
wording) comes from `USER_WORKFLOW.md` → `## Commits`.

## Scope reporting

When a commit gets no scope because none in `USER_WORKFLOW.md` → `### Scopes` fits, the skill
says so in its report with a suggested new scope, and in unattended mode adds the line
`SCOPE: <short commit> none — suggest <scope> (<what it would cover>)` before its `RESULT:`
line. Orchestrators collect these into their final report. Never add the scope to the list
yourself.

# Part 2: the files and their entry formats

## The files

All at the repo root.

| File | Written by | Worked off by |
|---|---|---|
| `CLAUDE_ANALYSIS.md` | `/analyse-code` | `/fix-next-finding`, `/fix-all-findings` |
| `USER_ANALYSIS.md` | the user, by hand | the same (alone with the `user-only` argument) |
| `USER_IDEAS.md` | the user, by hand (and `/pitch-feature` or `/brainstorm-feature` with the user's approval) | `/plan-features`, which turns ideas into concrete features in `CLAUDE_PLAN.md` |
| `CLAUDE_IDEAS.md` | `/suggest-features` | `/pitch-feature`, which moves an idea the user approves into `USER_IDEAS.md` |
| `CLAUDE_PLAN.md` | `/plan-features`, from the user's ideas and answers (the user may edit it too) | `/implement-next-feature`, `/implement-all-features` |
| `USER_WORKFLOW.md` | the user (drafted by `/setup-workflow`) | read by every skill |

Templates for every file are in the `templates/` directory next to this file.

## Choosing a model

Skills that write work entries (`/analyse-code`, `/plan-features`) pick by
**difficulty, not severity**, and write `**Model:**` only when the pick differs from the
default for that kind of work:

- `haiku`: mechanical work with an exact recipe (renames, dead code, docs, formatting).
- `sonnet`: a clear task with a precise **Fix:** / **How:**, size S or M, few files.
- `opus`: design-heavy or cross-cutting work, bugs with an unclear cause, persisted-format
  changes, size L.

Add `reasoning high` only for items whose difficulty is mostly thinking (subtle ordering or
concurrency bugs, tricky algorithms).

## Analysis files (`CLAUDE_ANALYSIS.md`, `USER_ANALYSIS.md`)

Sections `## 1. Bugs`, `## 2. Performance`, `## 3. Cleanliness / architecture`,
`## 4. Suggested order of work`. Ids are `<section>.<n>`. Severity **[H]** (fix before building
more on top), **[M]**, **[L]** (nice to have). `CLAUDE_ANALYSIS.md` also has a **Passes:** list
in its header, one line per `/analyse-code` pass.

```markdown
### 1.10 [M] One-line statement of the problem
_Found: 2026-10-24 14:54 CEST (`894a42a`)_

`path/to/file:12-30`, `other/file:5`

What goes wrong and why, with a concrete scenario.

**Fix:** the suggested change.

**Model:** opus
```

Section 4 is the main guide for the order of work. It may bundle items
("**1.1** together with **3.2**").

## The plan (`CLAUDE_PLAN.md`)

The concrete features that get built: one section `## Concrete features` (`F<n>`, with its
`_Next id:_`), optionally followed by a suggested-order section. Every entry is specific
enough for one agent to build without asking: behaviour decided, **Done when:** agreed with
the user, size S or M (L only if it can't be split). Entries come from `/plan-features`,
which writes them from the user's ideas and answers; writing an entry there is the user's
approval to build it. The user may edit or add entries by hand: their **Notes for Claude:**
are binding like in the user's lists, and Claude keeps such edits.

Header: a **Passes:** list, one line per `/plan-features` run. Severity **[H]** most wanted,
**[M]** wanted, **[L]** nice to have, taken from the idea it came from (default **[M]**).
**[CONFLICT]** marks an item that contradicts the design docs; the user's decision to build it
anyway is in its **Decisions:**.

```markdown
### F3 [M] One-line name of the feature
_Written: 2026-10-24 14:54 CEST (`894a42a`)_

**Idea:** I4 (the idea or wish it came from, in a few words)

**What:** what the user of the product sees and does, in plain words.

**Why:** how it improves the product.

**Done when:** what counts as finished.

**Design docs:** which part of the design docs it builds on (file and section), or "none".

**How:** rough implementation sketch: systems/files involved, new data, persisted-format
changes. "Needs F2" if it builds on another feature.

**Size:** S / M / L, and risks.

**Decisions:** the user's answers and the small details chosen (2026-10-24)

**Keep open for:** I2 (multiplayer), "sorting by date later".

**Model:** sonnet
```

Required: **What**, **Why**, **Done when:**, **Design docs**, **How**, **Size**. Older entries
may call **Size** `**Effort:**`; read it the same way.

## Idea files (`USER_IDEAS.md`, `CLAUDE_IDEAS.md`)

One section `## Ideas` (`I<n>`, with its `_Next id:_`). **Ideas are a backlog, never work.**
No run, triage or implement skill picks, blocks or asks about them. They become work only
through `/plan-features`, which turns them into concrete features in `CLAUDE_PLAN.md` and
then deletes the idea (or narrows it to what is left, with `_Updated:_`). Ideas matter to
running work only through **Keep open for:**.

- **`USER_IDEAS.md`** is the user's. Only the heading is required (`### I3 Undo the last
  move`, or just a title); id, severity, stamp and text are optional, and an idea may be a
  single line. Claude adds entries only when the user approves one of its own ideas
  (`/pitch-feature`) or parks a talked-through idea (`/brainstorm-feature`), and otherwise
  edits only the idea being planned.
- **`CLAUDE_IDEAS.md`** is Claude's own suggestions (`/suggest-features`). Nothing in it is
  planned before the user approves it into `USER_IDEAS.md` (`/pitch-feature`) or names it
  explicitly (`/plan-features claude:I2`). Header: a **Passes:** list, one line per
  `/suggest-features` pass. Severity **[H]** big improvement of the core loop/product,
  **[M]** clear improvement, **[L]** nice to have. **[CONFLICT]** marks the rare idea that
  contradicts the design docs. At the end, a `## Dropped` section: ideas the user turned down,
  one line each, `- <id> <name> — <reason> (<date>)`, so later passes don't suggest them again
  (the only non-open items kept anywhere). It is written by `/pitch-feature` and
  `/plan-features`.

Claude's ideas use the stamp and **What**, **Why**, **Design docs**, **Open questions**
(required), optionally a rough **How:**. A `[CONFLICT]` idea adds, right after **What**:
**Conflicts with:** (file, section and quote, and how it contradicts) and **Why anyway:**.
