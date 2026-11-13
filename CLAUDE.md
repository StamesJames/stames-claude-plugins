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

## Workflow

This project uses the `dev-workflow` plugin's skills (`/analyse-code`, `/fix-next-finding`,
`/fix-all-findings`, `/suggest-features`, `/pitch-feature`, `/brainstorm-feature`,
`/plan-features`, `/implement-next-feature`, `/implement-all-features`, `/review-change`,
`/setup-workflow`). The running lists `CLAUDE_ANALYSIS.md`, `USER_ANALYSIS.md`,
`USER_IDEAS.md`, `CLAUDE_IDEAS.md` and `CLAUDE_PLAN.md` sit at the repo root; their rules are in
`list-format.md` of the `dev-workflow` plugin's `setup-workflow` skill. The user's
preferences for them (design docs, idea sources, code standards, hands-off paths, what to ask
before, run settings, commit rules and scopes) are in `USER_WORKFLOW.md`. This section holds
the facts; keep it current when the code changes. The lists may be newer than this file: if
they disagree, check the code. When a finding or plan item is resolved during normal work,
delete its entry in the same change.

The root list files and `USER_WORKFLOW.md` are this repo's own workflow files. The files
under `plugins/dev-workflow/skills/setup-workflow/templates/` are plugin code (what
`/setup-workflow` copies into other projects); don't confuse the two. The skills run from
the installed copy in the plugin cache, so changes here take effect only after a version
bump and a marketplace update.

### Code scope

- Project code: `plugins/` (skill prompts, shared rule files, templates, `bin/workflow-lists`,
  `hooks/hooks.json`, manifests, plugin READMEs), plus `.claude-plugin/marketplace.json`,
  `README.md` and `CLAUDE.md`.
- Generated / not project code: none. (Hands-off paths are in `USER_WORKFLOW.md`.)
- List paths are relative to the repo root.

### Verification

How to check a change; every fix and feature runs these.

1. `claude plugin validate .`, then `claude plugin validate plugins/<name>` for each changed
   plugin: all print `✔ Validation passed`. Known, accepted warning: the
   `plugins/dev-workflow/README.md` install-line hint; no new warnings.
2. A changed plugin has its `version` bumped in `plugin.json` (patch for fixes and wording,
   minor for features, major for breaking list-format changes).
3. Cross-references resolve (prints only `done`):
   ```sh
   for p in plugins/*/; do grep -rhoE '\$\{CLAUDE_PLUGIN_ROOT\}/[A-Za-z0-9_./-]+' "$p" | sort -u | sed 's/[.]$//' | while read r; do f="$p${r#\$\{CLAUDE_PLUGIN_ROOT\}/}"; [ -e "$f" ] || echo "missing: $f"; done; done; grep -rhoE 'dev-workflow:[a-z-]+' plugins | sort -u | while read s; do [ -d "plugins/dev-workflow/skills/${s#dev-workflow:}" ] || echo "missing skill: $s"; done; echo done
   ```
4. If `bin/workflow-lists`, the templates or the list rules changed, the smoke test against
   fresh templates (prints `OK: no problems …`); if the `run` commands changed, also drive
   a mock run there (`WORKFLOW_LISTS_CLAUDE=<mock> workflow-lists run go features` after
   `run start` / `run queue`, with a mock that commits and prints the result JSON) and
   check the commits, `workflow/failed/*` branches, report and stats file:
   ```sh
   S=/tmp/claude-$(id -u)/wl-smoke; rm -rf $S; mkdir -p $S && cd $S && git init -q && T=<repo>/plugins/dev-workflow/skills/setup-workflow/templates && cp $T/{CLAUDE_ANALYSIS,USER_ANALYSIS,CLAUDE_PLAN,USER_IDEAS,CLAUDE_IDEAS,USER_WORKFLOW}.md . && cp $T/claude-md-section.md CLAUDE.md && python3 -I <repo>/plugins/dev-workflow/bin/workflow-lists check
   ```
   Also run `python3 -I plugins/dev-workflow/bin/workflow-lists check` at the repo root
   (this repo's own lists) and check that changed rules still match what the script checks.

These checks don't run the skills. How a changed prompt behaves only shows once the updated
plugin is installed and the skill is used in a target project, so the report says this.

## The `dev-workflow` plugin

Its skills share one workflow built around running list files at the
root of the *target* project (not this repo): `CLAUDE_ANALYSIS.md`, `USER_ANALYSIS.md`,
`USER_IDEAS.md`, `CLAUDE_IDEAS.md`, `CLAUDE_PLAN.md`, the user's `USER_WORKFLOW.md`, and a
`## Workflow` section in the target's `CLAUDE.md`.

How the pieces fit:

- **Four shared files in `skills/setup-workflow/`** hold the rules; skills read them instead
  of repeating them, so change rules there:
  - `list-format.md`: list format and rules (ids, `_Next id:_`, stamps, `[HOLD]`, "done
    means deleted", Ideas as backlog, `Keep open for:`, `Decisions:` / `Model:` /
    `_Failed:_` lines, model rubric, `Refs:` footers, scope reporting). Part 1 (up to
    `# Part 2`) holds what the skills that write or triage entries need; part 2 holds the
    files and entry formats. Keep that split.
  - `item-rules.md`: the digest of part 1 an agent working on one item needs (binding
    lines, `Verify:`, `Keep open for:`, `[HOLD]`, editing through the script, `Refs:`
    footers). Printed by `workflow-lists brief`; single-item skills read no part of
    `list-format.md`. A rule change in part 1 that concerns item agents goes here too.
  - `work-item.md`: the steps for one item (sections A–H: modes and the `brief` context,
    needs-the-user check, build, self-review, verify with the attempt budget, update list,
    commit, report and `RESULT:` lines). Its section letters are referenced by
    `fix-next-finding`, `implement-next-feature` and the driver's rework prompt (C–E, G);
    keep them stable or update those.
  - `loop.md`: the run protocol for the orchestrating session: up-front triage and
    decisions, then `workflow-lists run go` in the background, and what to do with its exit
    code. Everything unattended (one `claude -p` per item, booking, review and rework
    rounds, WIP branches, escalation, failure stamps, after-run analysis with parallel part
    agents, stats, report) is in the script. Changing `work-item.md`'s `RESULT:` /
    `BLOCKED:` / `DEFAULTS:` / `UNVERIFIED:` / `NOTE:` lines or arguments, or
    `review-change` (`ACCEPTED` / `REWORK` / `FAILED`) / `analyse-code` result lines, means
    updating `workflow-lists run` (`apply_result`, the prompts in `run go`) too.
- `skills/setup-workflow/templates/` holds the templates `/setup-workflow` copies into a target
  project (the five lists, `USER_WORKFLOW.md`, and `claude-md-section.md` for the target's
  `CLAUDE.md`). If a rule or section changes, check whether the templates and the "Check" mode
  of `setup-workflow/SKILL.md` need the same change.
- **Single-item skills** (`fix-next-finding`, `implement-next-feature`) only hold what is
  specific to their lists (picking order, needs-the-user cases, commit type); the rest is
  `work-item.md`. `user-only` restricts the fix skills to `USER_ANALYSIS.md`.
- **Loop skills** (`fix-all-findings`, `implement-all-features`) are a few specifics on top of
  `loop.md`. They never touch code. `review-change` is the acceptance gate of a run: it
  re-runs the verification and the entry's `Verify:` steps, sends an unmet item back
  (`REWORK`, up to `Rework rounds` from `## Runs`, the last round reviewed with `final`),
  then tidies.
- **Feature flow:** the user writes ideas (as vague as they like) in `USER_IDEAS.md`;
  `plan-features` asks the open questions and writes concrete features into `CLAUDE_PLAN.md`
  (an entry there is approved for building) and deletes the planned ideas; only
  `CLAUDE_PLAN.md` is implemented. `brainstorm-feature` talks a rough idea through first and
  hands the summary to `plan-features` as a free-text wish, skipping `USER_IDEAS.md`.
  Claude's own ideas go to `CLAUDE_IDEAS.md` (`suggest-features`) and reach the plan only
  through `pitch-feature` (moves one into `USER_IDEAS.md`) or an explicit
  `plan-features claude:I<n>`. Ideas are never work.
- `setup-workflow` section C migrates the pre-3.0 layout (`USER_PLAN.md`); `workflow-lists`
  flags that layout. Keep both if the list layout changes again.
- `review-change` and `analyse-code since=<commit> after-run` are also run by the driver,
  unattended. `analyse-code/part.md` holds the analysis checks and the read-only part-agent
  protocol; large after-run analyses are split by the driver into parallel part agents,
  with part findings written to `$(git rev-parse --git-path workflow-run)/analysis/`, next to
  the run log and state (`<kind>.log`, `<kind>.json`) that `workflow-lists run` keeps there.
- **Token budget:** skills read only the parts of files they need (`workflow-lists brief` /
  `show` / `summary` / `find` / `section`), use the `CLAUDE.md` already in their context
  instead of reading it, and edit lists through the script instead of Read/Edit. The
  headings these calls name are load-bearing: `USER_WORKFLOW.md` → `Design docs`,
  `Code standards`, `Hands-off`, `Ask before`, `Commits`, `Analysis focus`, `Runs`;
  `list-format.md` → `# Part 2`, `Commit footer`, `Scope reporting`, `Analysis files`,
  `The plan`, `Idea files`, `Choosing a model`; `CLAUDE.md` → `## Workflow` with
  `### Code scope` and `### Verification`; `CLAUDE_IDEAS.md` → `Dropped`. Renaming one means
  updating the calls (`brief` reads the `USER_WORKFLOW.md` and `CLAUDE.md` ones).
- **`bin/workflow-lists`** (Python 3, stdlib only) does everything mechanical about the lists
  and runs: reading (`check`, `summary`, `show`, `find`, `section`, `brief`, `stamp`),
  editing (`add`, `delete`, `set`, `move`, `pass`: ids, stamps, mentions,
  `_No open items._`, failed branches), the run order and mechanical triage (`triage`), the
  run state (`run start|queue|skip|log|next|result|commit|changes|end`), the **driver**
  (`run go`: one `claude -p --output-format json` per item with model, effort,
  `--max-turns` and `--max-budget-usd` from `## Runs`, `CLAUDECODE` unset so it runs from
  inside a session; `--auto` prepares a run without the skill, queuing only `READY` items)
  and the **stats** (`stats`; one JSON line per agent and run in the `Stats file`, with the
  plugin version from `plugin.json`, model, size, turns, tokens by cache type, wall-clock
  and model (API) time, cost, outcome). `WORKFLOW_LISTS_CLAUDE=<path>` substitutes the
  `claude` binary (a mock in tests). It parses suggested-order steps (`1. **1.1** + **3.2**: why`) and
  `**Keep open for:**` lines in the format `list-format.md` gives, and prints what it can't
  edit as `file:line`. `hooks/hooks.json` runs `workflow-lists hook` after `Write`/`Edit` of
  a list file. When a list rule, section name, entry field, template or `USER_WORKFLOW.md`
  → `## Runs` setting changes, update the script too (it reads the `USER_WORKFLOW.md`
  template itself), then run `workflow-lists check` against fresh templates in a scratch git
  repo: it must print `OK`. A change to the editing or `run` commands is tested the same
  way: add entries, then delete / set / drive them with a mock `claude` in the scratch repo
  and check the files, branches and the output.

Cross-skill references use `${CLAUDE_PLUGIN_ROOT}/skills/...` paths and the
`dev-workflow:<skill>` skill names; keep both in sync when renaming or moving skills.

## The `agent-context` plugin

One skill, `tidy-claude-md`, holding the whole convention: CLAUDE.md under 200 lines, path rules
in `.claude/rules/`, on-demand `CLAUDE_<TOPIC>.md` files referenced (never `@`-imported) from a
`## Context files` index. It only manages files in that index, so `dev-workflow`'s lists
(`CLAUDE_ANALYSIS.md`, `CLAUDE_PLAN.md`, `CLAUDE_IDEAS.md`) and its `## Workflow` section stay
untouched; keep it that way if either plugin changes its file names or sections. Scope is
CLAUDE.md and its linked files only, not skills or agent definitions.
