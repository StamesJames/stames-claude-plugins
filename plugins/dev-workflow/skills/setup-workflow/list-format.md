# Workflow lists: shared format and rules

Read by every workflow skill. A project uses this workflow when it has a `USER_WORKFLOW.md` at
its root. Two more shared files sit next to this one: `work-item.md` (how one item is built,
verified and committed) and `loop.md` (how a run works off a whole list).

## The files

All at the repo root.

| File | Written by | Worked off by |
|---|---|---|
| `CLAUDE_ANALYSIS.md` | `/analyse-code` | `/fix-next-finding`, `/fix-all-findings` |
| `USER_ANALYSIS.md` | the user, by hand | the same (alone with the `user-only` argument) |
| `CLAUDE_PLAN.md` | `/plan-features` | nothing directly: items are approved into `USER_PLAN.md` (`/pitch-feature`, `/explore-feature`), or `/pitch-feature` builds one right away |
| `USER_PLAN.md` | the user by hand, and Claude with the user's approval (`/explore-feature`, `/pitch-feature`) | `/implement-next-feature`, `/implement-all-features` |
| `USER_WORKFLOW.md` | the user (drafted by `/setup-workflow`) | read by every skill |

`CLAUDE.md` holds the **facts** the skills need (`## Workflow` → `### Code scope`,
`### Verification`, plus the architecture, conventions and invariants it documents anyway);
Claude keeps it current. `USER_WORKFLOW.md` holds the user's **decisions and preferences**
(design docs, idea sources, analysis focus, code standards, hands-off paths, what to ask
before, run settings, commit rules); Claude edits it only when the user asks. If the two
disagree, `USER_WORKFLOW.md` wins on preferences and the code wins on facts. The lists may be
newer than `CLAUDE.md`; if they disagree, check the code.

Templates for every file are in the `templates/` directory next to this file.

## Rules for all four lists

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
  (plans). Take the values from `date '+%Y-%m-%d %H:%M %Z'` and `git rev-parse --short HEAD`
  (`no commit` if the repo has none yet); never guess the time. Keep the stamps of other
  entries as they are.
- **[HOLD]** after the severity: the item is not touched (fixed, implemented, pitched) until
  the user removes the tag, even when named explicitly. Held items are not "blocked" and don't
  count as open for `NOTHING_LEFT`. A pass may still delete a held item that turns out done.
- **The user's entries** (`USER_*`): **Notes for Claude:** are binding constraints, **Done
  when:** is the acceptance criterion. Entries are often short (a symptom, no paths, no fix):
  that is not a reason to skip them. Edit only the entry being resolved and its mentions;
  leave everything else exactly as the user wrote it. The only lines Claude adds to a user
  entry it isn't resolving are the run lines below (`**Decisions:**`, `_Failed:_`).

## Optional entry lines (all lists)

These may appear in any entry, right before the closing text or at its end:

- `**Decisions:** <the user's answers> (<date>)`: decisions the user made for this item,
  recorded by a run's decision step (`loop.md`) or by `/explore-feature`. Binding like
  **Notes for Claude:**; an item with decisions covering its open questions no longer needs
  the user.
- `**Model:** <model>[, reasoning <level>]`: which model (and reasoning effort) a run uses
  for this item, overriding the defaults in `USER_WORKFLOW.md` → `## Runs`. Written only when
  it differs from that default. Models: `haiku`, `sonnet`, `opus` (or any model the Agent tool
  accepts); levels: `low`, `medium`, `high`, `xhigh`.
- `_Failed: <date time TZ> (`<wip branch>`, <model>): <one-line reason>_` under the
  `_Found:_` / `_Written:_` line: a run tried the item and failed; the attempt is saved on the
  named branch. Written only by `loop.md`. Such an item needs the user's decision before the
  next run touches it; the decision step turns the stamp into a `**Decisions:**` line that
  still names the branch.

When an entry is deleted as done, also delete the `workflow/failed/...` branches it names
(`git branch -D`), in the same run.

### Choosing a model

Skills that write entries (`/analyse-code`, `/plan-features`, `/explore-feature`) pick by
**difficulty, not severity**, and write `**Model:**` only when the pick differs from the
default for that kind of work:

- `haiku`: mechanical work with an exact recipe (renames, dead code, docs, formatting).
- `sonnet`: a clear task with a precise **Fix:** / **How:**, size S or M, few files.
- `opus`: design-heavy or cross-cutting work, bugs with an unclear cause, persisted-format
  changes, size L.

Add `reasoning high` only for items whose difficulty is mostly thinking (subtle ordering or
concurrency bugs, tricky algorithms).

## Ids outside the files

In arguments, `skip=` lists, reports, `RESULT:` lines, branch names and commit footers:

- `CLAUDE_ANALYSIS.md` items: bare id (`1.4`); in commits `analysis 1.4`; WIP branch
  `workflow/failed/analysis-1.4`.
- `USER_ANALYSIS.md` items: `U` prefix (`U1.1` = item `1.1` there); in commits
  `user analysis 1.1`; WIP branch `workflow/failed/user-analysis-1.1`.
- `USER_PLAN.md` items: their heading id (`F2`, `I1`; a heading without one goes by a short
  slug of its title); in commits `user plan F2`; WIP branch `workflow/failed/user-plan-F2`.
- `CLAUDE_PLAN.md` items: `claude:F3` / `claude:I2` wherever a `USER_PLAN.md` id could also be
  meant; in commits `claude plan F3`.

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

## Plan files (`CLAUDE_PLAN.md`, `USER_PLAN.md`)

Sections `## Concrete features` (`F<n>`, specific enough to build now) and `## Ideas` (`I<n>`),
each with its own `_Next id:_`. `USER_PLAN.md` may have a suggested-order section if the user
writes one.

**Ideas are a backlog, never work.** They hold vague ideas so they aren't lost. No run, triage
or implement skill picks, blocks or asks about them; they only become work through
`/explore-feature`, which turns one into a concrete feature with a new `F` id (deleting the
idea, or narrowing it to what is left). Ideas matter to running work only through
**Keep open for:** (below).

`CLAUDE_PLAN.md` also has a **Passes:** list in its header, and a `## Dropped` section at the
end: items the user turned down, one line each, `- <id> <name> — <reason> (<date>)`, so later
passes don't suggest them again (the only non-open items kept anywhere). It is written by
`/pitch-feature` and `/explore-feature`. Severity there is **[H]** big improvement of the core
loop/product, **[M]** clear improvement, **[L]** nice to have. **[CONFLICT]** after the
severity marks the rare item that contradicts the design docs.

```markdown
### F3 [M] One-line name of the feature
_Written: 2026-10-24 14:54 CEST (`894a42a`)_

**What:** what the user of the product sees and does, in plain words.

**Why:** how it improves the product.

**Done when:** what counts as finished.

**Design docs:** which part of the design docs it builds on (file and section), or "none".

**How:** rough implementation sketch: systems/files involved, new data, persisted-format
changes.

**Size:** S / M / L, and risks.

**Keep open for:** I2 (multiplayer), "sorting by date later".

**Model:** sonnet
```

Only the heading and the stamp are required in `USER_PLAN.md`; Claude writes the full format.
Older entries may call **Size** `**Effort:**`; read it the same way.

**Keep open for:** names ideas (`I2`, `claude:I4`) or free-text future needs the
implementation must **not block**: avoid choices that would make them expensive later (a
hard-coded single instance where a list is planned, a format that can't be extended). It does
**not** mean building for them: no speculative abstractions, hooks, options or config for
features that don't exist yet.

Ideas use the same stamp and **What/Why/Design docs**, with **Open questions** instead of the
rest. A `[CONFLICT]` item adds, right after **What**: **Conflicts with:** (file, section and
quote, and how it contradicts) and **Why anyway:**.

## Commit footer

Every commit that resolves (or partly resolves) a list item ends its message, before the
attribution lines, with one `Refs:` line per item, carrying the item's title so
`git log --grep 'Refs:' --oneline` reads as a log of finished work:

```
Refs: analysis 1.4 (money label not updated on load)
Refs: user plan F2 (pause menu sound)
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
