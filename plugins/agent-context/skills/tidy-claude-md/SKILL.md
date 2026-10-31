---
name: tidy-claude-md
description: Tidy a project's CLAUDE.md and the agent context files it links to (on-demand CLAUDE_<TOPIC>.md files listed in its "Context files" index, and path-scoped .claude/rules/ files) - keep CLAUDE.md under 200 lines, remove stale, duplicated or obvious content, and move detail out to the right tier. Proposes the changes and applies them only after approval; `check` only reports. Use when the user asks to tidy, trim, shrink, clean up, restructure or audit CLAUDE.md or its split-out context files. Not for skills or agent definitions.
---

# Tidy CLAUDE.md

CLAUDE.md is loaded into every session, so every line costs context in every session, and the
longer it gets, the less reliably it is followed. This skill keeps it lean by putting each piece
of content in the cheapest place where agents will still find it when they need it.

Arguments (all optional):

- `check`: report only, change nothing.
- a path: tidy that CLAUDE.md (e.g. a nested `packages/api/CLAUDE.md`) instead of the root one.

## The three tiers

| Tier | Where | Loaded | Limit |
|---|---|---|---|
| Always | `CLAUDE.md` | every session | under 200 lines |
| By path | `.claude/rules/<topic>.md` with `paths:` frontmatter | automatically, when Claude works on matching files | ~100 lines |
| On demand | `CLAUDE_<TOPIC>.md` next to the CLAUDE.md, listed in its `## Context files` index | only when an agent decides to read it | ~400 lines |

Where content belongs:

- **Always**: what nearly every task needs - what the project is, build/test/run commands,
  the layout in a few lines, hard rules that apply everywhere, gotchas that bite anywhere.
- **By path**: rules and conventions tied to part of the code ("handlers in `src/api/` must
  ..."). Path rules load without being asked, so use them for things that must not be missed
  when touching those files.
- **On demand**: reference material a task *may* need - a full architecture description, a
  subsystem deep dive, data formats, a domain glossary, how to do a rare task (release,
  migration).
- **Nowhere**: what the code, `git log` or `--help` already say, generic best practice, one-off
  fixes that will not recur, and content that is stale.

Limits are soft: a file over its limit is flagged and a split by topic is proposed, never done
mechanically (no "part 1 / part 2" splits).

## On-demand files and the index

On-demand files are named `CLAUDE_<TOPIC>.md` in upper case (`CLAUDE_ARCHITECTURE.md`,
`CLAUDE_SAVE_SYSTEM.md`) and sit next to the CLAUDE.md that lists them. They are **referenced,
never imported**: no `@CLAUDE_<TOPIC>.md`, since an import loads the file into every session
and saves nothing.

The CLAUDE.md lists them in a `## Context files` section, one line per file saying **when** to
read it, so an agent can match it to its task:

```markdown
## Context files

Read these when the task matches; they are not loaded automatically.

- `CLAUDE_ARCHITECTURE.md`: before adding modules or changing how components talk to each other.
- `CLAUDE_SAVE_SYSTEM.md`: when touching save/load, the save format or migrations.
```

"Architecture: see `CLAUDE_ARCHITECTURE.md`" is not enough; the trigger is what makes agents
read it. Each on-demand file starts with a one-line summary of what it covers.

**This skill only manages files listed in the index.** Other `CLAUDE_*.md` files (for example
the `dev-workflow` plugin's running lists `CLAUDE_ANALYSIS.md` and `CLAUDE_PLAN.md`) belong to
something else: never edit, move or list them. The same goes for sections of CLAUDE.md that other
tooling maintains, such as the `## Workflow` section written by `dev-workflow`'s
`/setup-workflow`: leave them in place and unchanged. If one looks stale or too long, say so in
the report and name the tool that owns it.

## 1. Inventory

1. Read the target CLAUDE.md in full, everything it imports with `@path` (imports count towards
   what loads every session), the files in its `## Context files` index, and
   `.claude/rules/*.md` (note each file's `paths:`).
2. Line counts for each against its tier's limit, plus the total that loads every session
   (CLAUDE.md + its imports + rules without `paths:`).
3. Note problems in the index itself: listed files that do not exist, entries with no "when"
   trigger, `CLAUDE_<TOPIC>.md` files that are mentioned in CLAUDE.md but missing from the index,
   and `@` imports of on-demand files.

`CLAUDE.local.md` and `~/.claude/CLAUDE.md` are out of scope unless the user names them.

## 2. Assess the content

Go through the content section by section and check it against the project, not just read it:

- **Stale**: commands, paths, file names, modules or behaviour that no longer match the code.
  Check: does the path exist, is the script in the build file, does the described structure
  still hold? Stale content is the worst kind, since agents act on it.
- **Duplicated**: the same fact in CLAUDE.md and an on-demand or rules file, or twice in one
  file. Keep it in one place, in the cheapest tier that still works.
- **Obvious or generic**: things an agent learns from the code in seconds, or advice that
  applies to every project ("write clean code", "add tests").
- **Verbose**: paragraphs that could be a line; explanations of *why* where only the rule is
  needed (keep the why when it prevents a wrong call).
- **Wrong tier**: detail in CLAUDE.md that only some tasks need (move to a rules file or an
  on-demand file); rules in an on-demand file that must never be missed (move to a rules file
  or CLAUDE.md); an on-demand file covering several unrelated topics (split by topic).
- **Conflicting**: two places that say different things. Do not pick a side if the code does
  not decide it: ask.

Never drop a hard rule, a non-obvious gotcha or the user's own instructions just to save lines.
If something looks unnecessary but you are unsure why it is there, keep it and ask in the
report.

## 3. Report and propose

Report first, change nothing yet:

```
## CLAUDE.md tidy report

Loads every session: <n> lines (CLAUDE.md <n> + imports <n> + unscoped rules <n>), target < 200.

| File | Tier | Lines | Limit | Issues |
|---|---|---|---|---|
| CLAUDE.md | always | 243 | 200 | 2 stale, 1 duplicated, 3 sections in wrong tier |
| CLAUDE_ARCHITECTURE.md | on demand | 180 | 400 | module list out of date |

### Proposed changes
1. <file>: <what changes and why>, -<n> lines.
2. Move "<section>" from CLAUDE.md to `CLAUDE_<TOPIC>.md` (new), index entry: "<when to read>".
3. ...

After: CLAUDE.md <n> lines, loads every session <n> lines.

### Questions
- <anything unclear: content whose purpose is unknown, conflicts the code does not decide>
```

Show the exact new text for index entries and for anything rewritten (not moved). For moves,
naming the section is enough. Group trivial removals into one item.

With `check`, stop here.

Otherwise ask which changes to apply (all, a selection, or none) and get the answers to the
questions in the same round.

## 4. Apply

1. Make the approved changes. Moving content means removing it from the source in the same
   change, so nothing ends up in two places.
2. New on-demand files: `CLAUDE_<TOPIC>.md` with a one-line summary at the top; add their
   index entries. Create the `## Context files` section if it is missing, near the end of
   CLAUDE.md, before sections owned by other tooling.
3. New rules files: `.claude/rules/<topic>.md` with `paths:` frontmatter listing the globs.
4. Re-read the changed files and recount: no broken references, every on-demand file is in
   the index, no on-demand file is imported, CLAUDE.md is under its limit (or say why not).

Do not commit; the user decides when.

## 5. Report

Before/after line counts per file and for what loads every session, the files created, moved
or deleted, what was left as is and why (owned by other tooling, unanswered questions), and
anything still over its limit.
