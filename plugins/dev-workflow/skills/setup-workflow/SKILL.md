---
name: setup-workflow
description: Set up the analysis/plan workflow (running lists CLAUDE_ANALYSIS.md, USER_ANALYSIS.md, CLAUDE_PLAN.md, USER_PLAN.md, the user's USER_WORKFLOW.md and a Workflow section in CLAUDE.md) in the current project, or, if it is already set up, check that configuration against the project and propose updates. Use when the user asks to set up, initialise, migrate or re-check the workflow, or when another workflow skill reports that USER_WORKFLOW.md is missing.
---

# Set up or check the workflow

The workflow skills (`analyse-code`, `fix-next-finding`, `fix-all-findings`,
`plan-features`, `explore-feature`, `pitch-feature`, `implement-next-feature`,
`implement-all-features`, `review-change`) are generic. A project plugs into them with:

- **`USER_WORKFLOW.md`** (repo root): the user's decisions and preferences. The user edits it
  by hand; you draft it here and afterwards change it only when asked.
- **`## Workflow` in `CLAUDE.md`**: facts, `### Code scope` and `### Verification`. You keep it
  current.
- **The four list files** at the repo root.

Read `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md` first.
Templates are in `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/templates/`.

This skill only writes the workflow files and `CLAUDE.md`. It never commits (the user commits
it) and never touches project code.

**Mode:** if `USER_WORKFLOW.md` exists, run **Check** (section B). Otherwise run **Set up**
(section A). The argument `check` forces B; `migrate` means A for a project that already has
an older, project-specific form of this workflow (see A.4).

## A. Set up

### A.1 Look at the project

1. `git rev-parse --is-inside-work-tree` and `git status --short`. Not a git repo: tell the
   user the workflow relies on git (stamps, commits) and ask whether to `git init` or stop.
2. Find out what the project is: languages, frameworks, entry points, the top-level layout.
   Read `CLAUDE.md` and `README*` if present. If there is no `CLAUDE.md`, suggest running
   `/init` first; if the user declines, you will create one holding only the Workflow section.
3. **Verification:** how is a change checked here? Look for test runners, linters, type
   checkers, build commands (`package.json` scripts, `Makefile`, `Cargo.toml`, `pyproject.toml`,
   `project.godot`, CI configs under `.github/workflows/` and similar), and any diagnostic
   scripts. Prefer what CI runs. Try the cheap ones to see that they work.
4. **Code scope:** which directories hold project code, which are generated (build output,
   caches, lock files) or vendored (`vendor/`, `third_party/`, `addons/`, copied libraries;
   check `.gitignore` and license files).
5. **Design docs:** `docs/`, `planning/`, `design/`, ADRs, specs, a detailed README.
6. **Areas** for the initial scope list: the main subsystems or modules, as a user would
   name them (5–12, short lowercase words). Add `workflow` for the list files and
   `USER_WORKFLOW.md` if the project has no better home for them.
7. `git log --oneline -30`, only to notice an existing commit convention that differs from the
   default; never adopt it silently.

### A.2 Draft and confirm

Draft `USER_WORKFLOW.md` from the template and the `## Workflow` section from
`templates/claude-md-section.md`, filled with what you found. Keep the template's `## Commits`
section as it is (it is the user's default) apart from the scope list. Show the user a short
summary of the draft: design docs, idea sources, analysis focus, code standards, hands-off
paths, ask-before rules, run settings (models), scopes, verification commands. Then ask with `AskUserQuestion` only what you could not
settle yourself, for example:

- verification commands, if none were found or they don't run;
- which docs are the design intent, and whether conflicting ideas are allowed;
- the scope list (offer your list as the recommended option);
- an existing commit convention that differs from the default: keep the default (recommended)
  or describe the existing one in `## Commits`.

Idea sources, analysis focus, code standards and run settings are drafts the user is
expected to edit later: propose sensible ones (code standards from the linters, formatter
configs and conventions you found) and say so instead of asking about each.

### A.3 Write

- `USER_WORKFLOW.md`: the confirmed draft. Remove the template's `<!-- -->` hints that are
  filled in; keep the rest of the explanatory text.
- The four list files from the templates, `<code root>` replaced. **Never overwrite an
  existing list file**; if one exists, leave it and check it in B.3 instead.
- `CLAUDE.md`: add the `## Workflow` section (before `## Architecture` or similar if the file
  has one, otherwise at the end). If a section with the same purpose exists, merge into it.

### A.4 Migrating an older, project-specific form

Typical signs: project-local copies of these skills in `.claude/skills/`, `CLAUDE.md` sections
like "Code analysis passes" or "Feature plans" that repeat the list rules, list headers that
point to `.claude/skills/...` or `~/.claude/skills/...` (from before these skills were the
`dev-workflow` plugin).

- Move preferences out of `CLAUDE.md` into `USER_WORKFLOW.md`: design docs, idea sources and
  analysis focus written in the old skills, "leave this alone" rules, what needs the user's
  OK, commit style. Facts (commands, scope, invariants) stay in `CLAUDE.md`; the verification
  steps from the old skills go into `### Verification`.
- Replace the old workflow sections in `CLAUDE.md` with the new `## Workflow` section, and drop
  the shared list rules it repeats (they are in `list-format.md` now).
- Update the headers of the existing list files to the templates' wording (skill paths, rule
  pointer) without touching their items, `_Next id:_` lines, Passes or Dropped lines.
- List the project-local skill copies and tell the user to delete them (they would shadow or
  duplicate the global ones). Don't delete them yourself unless asked.

### A.5 Report

What you created or changed, what the user should review first in `USER_WORKFLOW.md` (idea
sources, analysis focus, code standards, run settings, scopes), the verification commands and whether they ran, and a
suggested commit message (`chore(workflow): set up workflow lists` or per the project's
`## Commits`). Remind the user that nothing was committed.

## B. Check

1. Read `USER_WORKFLOW.md`, the `## Workflow` section of `CLAUDE.md` and the headers of the
   four list files. Compare `USER_WORKFLOW.md` with the current template: report sections the
   template has and the file lacks (the template may have grown).
2. Check the configuration against the project:
   - paths in design docs, hands-off, code scope that no longer exist, and new top-level
     directories that none of them covers;
   - verification commands that fail, are missing, or no longer match the build/test setup
     (run the cheap ones);
   - scopes that match no area any more, and areas that recent commits (`git log -50`) or the
     code show without a scope;
   - list files that are missing, lack `_Next id:_` lines, or have headers pointing to old
     skill paths or naming skills that no longer exist (`/fix-next-user-finding`,
     `/fix-all-user-findings`, `/implement-all-claude-features`: now `user-only` arguments,
     and `CLAUDE_PLAN.md` items are approved into `USER_PLAN.md`); the same for the
     `## Workflow` section of `CLAUDE.md`.
3. Fix facts directly: the `## Workflow` section in `CLAUDE.md`, missing list files (from the
   templates), list headers. For `USER_WORKFLOW.md`, **propose** the changes and apply only
   those the user approves (`AskUserQuestion`, multi-select where it fits).
4. Report what you changed, what you proposed and what was approved. Don't commit.
