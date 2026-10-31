# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal Claude Code plugin marketplace (`stames-plugins`). There is no build, no tests and
no code to run: the "code" is Markdown skill prompts plus JSON manifests.

- `.claude-plugin/marketplace.json`: the marketplace; one entry per plugin with
  `"source": "./plugins/<name>"`.
- `plugins/<name>/.claude-plugin/plugin.json`: each plugin's manifest; its `name` must match
  the marketplace entry.
- `plugins/<name>/skills/<skill>/SKILL.md`: the skills (frontmatter `name` + `description`;
  the description is what triggers the skill, so keep it precise).

## Commands

```sh
claude plugin validate .                    # validate the marketplace
claude plugin validate plugins/<name>       # validate one plugin
```

Run both after any change to manifests or skills. **Bump `version` in the plugin's
`plugin.json` whenever you change the plugin**, or installs won't pick up the update.

Adding a plugin: create `plugins/<name>/` with `.claude-plugin/plugin.json`, add its entry to
`marketplace.json`, add a row to the table in `README.md`, then validate.

## The `dev-workflow` plugin

Its skills share one workflow built around running list files at the
root of the *target* project (not this repo): `CLAUDE_ANALYSIS.md`, `USER_ANALYSIS.md`,
`CLAUDE_PLAN.md`, `USER_PLAN.md`, the user's `USER_WORKFLOW.md`, and a `## Workflow` section in
the target's `CLAUDE.md`.

How the pieces fit:

- **Three shared files in `skills/setup-workflow/`** hold the rules; skills read them instead
  of repeating them, so change rules there:
  - `list-format.md`: list format and rules (ids, `_Next id:_`, stamps, `[HOLD]`, "done means
    deleted", Ideas as backlog, `Keep open for:`, `Decisions:` / `Model:` / `_Failed:_`
    lines, model rubric, `Refs:` footers, scope reporting).
  - `work-item.md`: the steps for one item (sections A–H: modes, needs-the-user check,
    build, self-review, verify, update list, commit, report and `RESULT:` lines). Its section
    letters are referenced by `fix-next-finding`, `implement-next-feature` and
    `pitch-feature`; keep them stable or update those skills.
  - `loop.md`: the run protocol (up-front triage and decisions, one subagent per item,
    review subagent, failure handling with WIP branches and model escalation, after-run
    analysis). Changing `work-item.md`'s `RESULT:` lines or arguments, or `review-change` /
    `analyse-code` result lines, means updating `loop.md` too.
- `skills/setup-workflow/templates/` holds the templates `/setup-workflow` copies into a target
  project (the four lists, `USER_WORKFLOW.md`, and `claude-md-section.md` for the target's
  `CLAUDE.md`). If a rule or section changes, check whether the templates and the "Check" mode
  of `setup-workflow/SKILL.md` need the same change.
- **Single-item skills** (`fix-next-finding`, `implement-next-feature`) only hold what is
  specific to their lists (picking order, needs-the-user cases, commit type); the rest is
  `work-item.md`. `user-only` restricts the fix skills to `USER_ANALYSIS.md`.
- **Loop skills** (`fix-all-findings`, `implement-all-features`) are a few specifics on top of
  `loop.md`. They never touch code.
- `CLAUDE_PLAN.md` is never worked off directly: `pitch-feature` builds one item or approves
  it into `USER_PLAN.md`, `explore-feature` works items out with the user and writes them to
  `USER_PLAN.md`. Only `USER_PLAN.md` concrete features are implemented by the loops.
- `review-change` and `analyse-code since=<commit>` are also run by the loops, unattended.
- **`bin/workflow-lists`** (Python 3, stdlib only) checks the mechanical part of
  `list-format.md` (`check`), summarises open items (`summary`) and prints stamps (`stamp`);
  `hooks/hooks.json` runs `workflow-lists hook` after `Write`/`Edit` of a list file. When a
  list rule, section name, entry field, template or `USER_WORKFLOW.md` → `## Runs` setting
  changes, update the script too (it reads the `USER_WORKFLOW.md` template itself), then
  run `workflow-lists check` against fresh templates in a scratch git repo: it must print `OK`.

Cross-skill references use `${CLAUDE_PLUGIN_ROOT}/skills/...` paths and the
`dev-workflow:<skill>` skill names; keep both in sync when renaming or moving skills.

## The `agent-context` plugin

One skill, `tidy-claude-md`, holding the whole convention: CLAUDE.md under 200 lines, path rules
in `.claude/rules/`, on-demand `CLAUDE_<TOPIC>.md` files referenced (never `@`-imported) from a
`## Context files` index. It only manages files in that index, so `dev-workflow`'s lists
(`CLAUDE_ANALYSIS.md`, `CLAUDE_PLAN.md`) and its `## Workflow` section stay untouched; keep it
that way if either plugin changes its file names or sections. Scope is CLAUDE.md and its linked
files only, not skills or agent definitions.
