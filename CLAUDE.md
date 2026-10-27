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

The only plugin so far. Its skills share one workflow built around running list files at the
root of the *target* project (not this repo): `CLAUDE_ANALYSIS.md`, `USER_ANALYSIS.md`,
`CLAUDE_PLAN.md`, `USER_PLAN.md`, the user's `USER_WORKFLOW.md`, and a `## Workflow` section in
the target's `CLAUDE.md`.

How the pieces fit:

- **`skills/setup-workflow/list-format.md` is the single source of truth** for the list format
  and rules (ids, `_Next id:_`, stamps, `[HOLD]`, "done means deleted", `Refs:` commit
  footers, scope reporting). Every other skill reads it instead of repeating it; change rules
  there, not in individual skills.
- `skills/setup-workflow/templates/` holds the templates `/setup-workflow` copies into a target
  project (the four lists, `USER_WORKFLOW.md`, and `claude-md-section.md` for the target's
  `CLAUDE.md`). If a rule or section changes, check whether the templates and the "Check" mode
  of `setup-workflow/SKILL.md` need the same change.
- **Single-item skills** (`fix-next-finding`, `implement-next-feature`) do the real work: pick,
  fix/implement, verify, delete the entry, commit. They support an `unattended` mode with
  `skip=<ids>` and emit machine-readable `RESULT:` / `SCOPE:` lines.
- **Loop skills** (`fix-all-findings`, `implement-all-features`) are orchestrators: they never
  touch code, they spawn one fresh `general-purpose` subagent per item that invokes the
  single-item skill unattended, and parse its `RESULT:` lines. Changing the single-item
  skill's argument or `RESULT:` format means updating the loop skill too.
- **`*-user-*` variants** (`fix-next-user-finding`, `fix-all-user-findings`) are thin wrappers
  that delegate to the general skill with the `user-only` argument; the logic lives only in
  the general skill.
- `CLAUDE_PLAN.md` is only built with the user's approval: `pitch-feature` implements one
  approved item, reusing steps of `implement-next-feature/SKILL.md` by number (keep those step
  numbers stable or update `pitch-feature`); `implement-all-claude-features` is a thin wrapper
  that runs `implement-all-features` with `claude-plan`, which asks the user to confirm the run
  and passes `claude-plan` on to `implement-next-feature`.

Cross-skill references use `${CLAUDE_PLUGIN_ROOT}/skills/...` paths and the
`dev-workflow:<skill>` skill names; keep both in sync when renaming or moving skills.
