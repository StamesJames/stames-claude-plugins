# Workflow

How Claude's workflow skills (`/analyse-code`, `/fix-next-finding`, `/suggest-features`,
`/pitch-feature`, `/brainstorm-feature`, `/plan-features`, `/implement-next-feature`,
`/review-change` and the loops `/fix-all-findings`, `/implement-all-features`) should work in
this project.
This file is yours: edit it by hand whenever you change your mind. Claude only edits it when
you ask. Facts about the code (verification commands, code scope, invariants) live in
`CLAUDE.md`, which Claude keeps current; list rules are in
`list-format.md` of the `dev-workflow` plugin's `setup-workflow` skill.

---

## Design docs

Where the intent of the project is written down. `/suggest-features` measures every idea
against these; `/plan-features` and the implement skills read them before planning and
building.

- `README.md`: what the marketplace is, which plugins it holds, how to install them.
- `plugins/dev-workflow/README.md`: what the `dev-workflow` plugin is for and how its skills
  fit together (plan-first, one fresh agent per item, lists checked mechanically).
- `CLAUDE.md` → "The `dev-workflow` plugin" and "The `agent-context` plugin": how each
  plugin is built and the conventions it keeps (shared rule files, feature flow, what
  `tidy-claude-md` may touch).

Claude does not edit these unless asked.

Conflicting ideas: allowed rarely, tagged [CONFLICT].

---

## Idea sources

Angles `/suggest-features` should look from for its own ideas, in rough order of importance.
Gaps against the design docs always come first.

- Friction in real runs: steps where a skill has to guess, asks too often, or leaves the
  lists in a state the next skill trips over.
- Unattended robustness: what makes a loop run stop, fail or produce a bad commit, and how
  the rules or `workflow-lists` could catch it earlier.
- Context economy: prompt text loaded that a skill doesn't need, rules that could move into
  a shared file or the script.
- Claude Code plugin features not used yet (agents, commands, more hooks, settings) where
  they would make a skill simpler or more reliable.
- Onboarding: what a new project or a new user needs to get going (READMEs, setup checks).

---

## Analysis focus

What `/analyse-code` should weigh most here, beyond the generic bugs / performance / cleanliness
checks. In this repo "code" is mostly skill prompts: a bug is an instruction that is wrong,
contradictory or ambiguous enough that an agent following it does the wrong thing.

- Cross-skill consistency: `${CLAUDE_PLUGIN_ROOT}/skills/...` paths, `dev-workflow:<skill>`
  names, `work-item.md` section letters, `RESULT:` / result lines between `work-item.md`,
  `review-change`, `analyse-code` and `loop.md`.
- Rules vs. script: `list-format.md`, the templates and `bin/workflow-lists` must agree
  (sections, fields, `## Runs` settings, ids, stamps).
- Unattended runs: instructions an agent could misread without a user to ask, missing
  failure paths, steps that could lose or duplicate list entries.
- Duplication: rules repeated in skills instead of read from the shared files.
- Skill descriptions: precise triggers that don't overlap between skills.

### Don't flag

Deliberate choices a pass should not report as problems.

- No build, no test suite: the only checks are `claude plugin validate`, the
  `workflow-lists` smoke test and the cross-reference grep.
- `bin/` makes `dev-workflow` Claude Code only (no Cowork / claude.ai install).
- `workflow-lists` uses the Python standard library only, even where a dependency would be
  shorter.
- The installed copy under `~/.claude*/plugins/cache/` lags behind this repo until the
  version is bumped and the marketplace updated.

---

## Code standards

What clean code means in this project. Every skill that writes code follows it, and
`/analyse-code` and `/review-change` measure against it. Conventions already documented in
`CLAUDE.md` don't need repeating.

- A rule lives in one place: shared rules in `setup-workflow`'s `list-format.md`,
  `work-item.md` or `loop.md`; skills point to them and hold only what is specific to them.
- Skill prompts are plain and short: imperative instructions, no filler, examples only where
  the format would otherwise be guessed.
- Markdown is wrapped like the surrounding file (around 95 characters).
- `workflow-lists`: standard library only, output of a few lines, exit codes as documented
  in its docstring.
- Tests: no test suite; a change to `workflow-lists`, the templates or list rules runs the
  scratch-repo smoke test from `CLAUDE.md` → `### Verification`.

---

## Hands-off paths

Never edited by any skill (vendored code, scratch files, assets you manage yourself). Not
reported as findings either.

- `LICENSE`: the license text.

---

## Ask before

Changes the skills must not make without your OK (runs ask about such items before they
start, all at once). These add to the generic rules: big refactors, items marked "verify",
design questions.

- Breaking changes to the list format or `USER_WORKFLOW.md` (they need a migration in
  `setup-workflow` and the script).
- Renaming, adding or removing a skill or a plugin.
- Changing hooks (`hooks/hooks.json`).
- Major version bumps.

---

## Runs

Settings for `/fix-all-findings` and `/implement-all-features` (one fresh agent per item). An
item's `**Model:**` line overrides the model for that item.

- Models: triage `opus`, features `opus`, fixes `sonnet`, review `sonnet`, analysis `sonnet`
  <!-- haiku | sonnet | opus | inherit (the session's model); add "reasoning <level>" after a
  model to set its effort, e.g. `opus, reasoning high` -->
- Escalate failed items to: `opus` <!-- retried once with this model before the item counts
  as failed; `none` to turn off -->
- Fresh review: features of size M or L, and any item whose diff has more than 150 changed
  lines <!-- or `always`, `never` -->
- Analyse after run: features yes, fixes no
- Stop after consecutive failures: 3
- Run branch: no <!-- yes: a run started on the default branch commits to
  `workflow/<date>-<kind>` instead. Failed attempts always go to `workflow/failed/<item>`. -->

---

## Commits

[Conventional Commits](https://www.conventionalcommits.org/): `type(scope)!: description`.

- Scope optional, from the list below. `!` marks a breaking change (e.g. an incompatible change
  of a saved or public format), explained in a `BREAKING CHANGE:` footer.
- Description: lowercase, imperative ("add", not "added"), no trailing period; subject line at
  most 72 characters. Optional body after a blank line: why, and anything not obvious from
  the diff.
- The type follows what the change does, not where it came from: a bug from the analysis list
  is `fix`, a cleanup finding is `refactor`, a performance finding is `perf`, a stale doc is
  `docs`.
- Workflow items are referenced in a footer (see `list-format.md`):
  `Refs: analysis 1.4 (<item title>)`.

Types:

- `feat`: new feature for the user
- `fix`: bug fixed in a user feature
- `docs`: documentation only
- `style`: formatting, no code change
- `refactor`: restructuring code without changing behaviour
- `perf`: performance improvement; the message says what got better (CPU, memory, GPU, load
  time) and by how much, measured where possible
- `test`: adding, refactoring or changing tests, no production code change
- `ci`: CI pipeline
- `build`: build scripts or tools
- `chore`: anything else (config, dependencies, the workflow list files)
- `revert`: reverting an earlier commit (`revert: <subject of the reverted commit>`, body
  `This reverts commit <hash>.`)

### Scopes

If no scope fits, commit without one and tell me in the answer, with a suggestion for a new
scope to add here.

- `dev-workflow`: the `dev-workflow` plugin (skills, shared rule files, templates,
  `workflow-lists`, hooks, its README and manifest)
- `agent-context`: the `agent-context` plugin
- `marketplace`: `.claude-plugin/marketplace.json`, the root `README.md`, `CLAUDE.md`
- `workflow`: this repo's own list files and `USER_WORKFLOW.md`
