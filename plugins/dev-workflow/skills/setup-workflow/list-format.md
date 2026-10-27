# Workflow lists: shared format and rules

Read by every workflow skill (`analyse-code`, `fix-*`, `plan-features`, `pitch-feature`,
`implement-*`, `setup-workflow`). A project uses this workflow when it has a
`USER_WORKFLOW.md` at its root.

## The files

All at the repo root.

| File | Written by | Worked off by |
|---|---|---|
| `CLAUDE_ANALYSIS.md` | `/analyse-code` | `/fix-next-finding`, `/fix-all-findings` |
| `USER_ANALYSIS.md` | the user, by hand | the same, plus `/fix-next-user-finding`, `/fix-all-user-findings` |
| `CLAUDE_PLAN.md` | `/plan-features` | `/pitch-feature` (one item after approval), or `/implement-all-claude-features` (only after the user confirms the run) |
| `USER_PLAN.md` | the user, by hand | `/implement-next-feature`, `/implement-all-features` |
| `USER_WORKFLOW.md` | the user (drafted by `/setup-workflow`) | read by every skill |

`CLAUDE.md` holds the **facts** the skills need (`## Workflow` → `### Code scope`,
`### Verification`, plus the architecture, conventions and invariants it documents anyway);
Claude keeps it current. `USER_WORKFLOW.md` holds the user's **decisions and preferences**
(design docs, idea sources, analysis focus, hands-off paths, what to ask before, commit rules);
Claude edits it only when the user asks. If the two disagree, `USER_WORKFLOW.md` wins on
preferences and the code wins on facts. The lists may be newer than `CLAUDE.md`; if they
disagree, check the code.

Templates for every file are in the `templates/` directory next to this file.

## Rules for all four lists

- **Running lists of open items only.** Never start a list from scratch; update it.
- **Done means deleted.** When an item is fixed or implemented, delete the whole entry, its
  mentions in a suggested-order section and its cross-references in other entries, in the same
  change. Don't strike through, don't mark as done. Git is the archive: the commit's `Refs:`
  footer (below) finds it again.
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
  leave everything else exactly as the user wrote it.

## Ids outside the files

In arguments, `skip=` lists, reports, `RESULT:` lines and commit footers:

- `CLAUDE_ANALYSIS.md` items: bare id (`1.4`); in commits `analysis 1.4`.
- `USER_ANALYSIS.md` items: `U` prefix (`U1.1` = item `1.1` there); in commits
  `user analysis 1.1`.
- `USER_PLAN.md` items: their heading id (`F2`, `I1`; a heading without one goes by a short
  slug of its title); in commits `user plan F2`.
- `CLAUDE_PLAN.md` items: `F3` / `I2`; in commits `claude plan F3`.

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
```

Section 4 is the main guide for the order of work. It may bundle items
("**1.1** together with **3.2**").

## Plan files (`CLAUDE_PLAN.md`, `USER_PLAN.md`)

Sections `## Concrete features` (`F<n>`, specific enough to build now) and `## Ideas` (`I<n>`,
still need design decisions), each with its own `_Next id:_`. An idea that becomes concrete
moves to Concrete features with a new `F` id and an `_Updated:_` stamp. `USER_PLAN.md` may have
a suggested-order section if the user writes one.

`CLAUDE_PLAN.md` also has a **Passes:** list in its header, and a `## Dropped` section at the
end: items the user turned down, one line each, `- <id> <name> — <reason> (<date>)`, so later
passes don't suggest them again (the only non-open items kept anywhere). Severity there is
**[H]** big improvement of the core loop/product, **[M]** clear improvement, **[L]** nice to
have. **[CONFLICT]** after the severity marks the rare item that contradicts the design docs.

```markdown
### F3 [M] One-line name of the feature
_Written: 2026-10-24 14:54 CEST (`894a42a`)_

**What:** what the user of the product sees and does, in plain words.

**Why:** how it improves the product.

**Design docs:** which part of the design docs it builds on (file and section), or "none".

**How:** rough implementation sketch: systems/files involved, new data, persisted-format
changes, open design choices.

**Effort:** S / M / L, and risks.
```

Ideas use the same stamp and **What/Why/Design docs**, with **Open questions** instead of
**How** and **Effort** if those aren't clear yet. A `[CONFLICT]` item adds, right after
**What**: **Conflicts with:** (file, section and quote, and how it contradicts) and **Why
anyway:**.

## Commit footer

Every commit that resolves (or partly resolves) a list item ends its message, before the
attribution lines, with one `Refs:` line per item, carrying the item's title so
`git log --grep 'Refs:' --oneline` reads as a log of finished work:

```
Refs: analysis 1.4 (money label not updated on load)
Refs: user plan F2 (pause menu sound)
```

Add `, partial` inside the parentheses for a partial fix. Everything else about the commit
message (types, scopes, wording) comes from `USER_WORKFLOW.md` → `## Commits`.

## Scope reporting

When a commit gets no scope because none in `USER_WORKFLOW.md` → `### Scopes` fits, the skill
says so in its report with a suggested new scope, and in unattended mode adds the line
`SCOPE: <short commit> none — suggest <scope> (<what it would cover>)` before its `RESULT:`
line. Orchestrators collect these into their final report. Never add the scope to the list
yourself.
