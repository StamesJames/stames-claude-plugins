# Workflow

How Claude's workflow skills (`/analyse-code`, `/fix-next-finding`, `/plan-features`,
`/explore-feature`, `/pitch-feature`, `/implement-next-feature`, `/review-change` and the
loops `/fix-all-findings`, `/implement-all-features`) should work in this project.
This file is yours: edit it by hand whenever you change your mind. Claude only edits it when
you ask. Facts about the code (verification commands, code scope, invariants) live in
`CLAUDE.md`, which Claude keeps current; list rules are in
`list-format.md` of the `dev-workflow` plugin's `setup-workflow` skill.

---

## Design docs

Where the intent of the project is written down. `/plan-features` measures every idea against
these; the implement skills read them before building.

<!-- Paths, and one line on what each holds. "None" if there are none. -->
- `<path>`: <what it holds>

Claude does not edit these unless asked.

Conflicting ideas: <allowed rarely, tagged [CONFLICT] | never>.

---

## Idea sources

Angles `/plan-features` should look from, in rough order of importance. Gaps against the design
docs always come first.

<!-- One line each, project-specific. -->
- <angle>: <what to look for>

---

## Analysis focus

What `/analyse-code` should weigh most here, beyond the generic bugs / performance / cleanliness
checks.

- <area>: <what matters, e.g. "must scale to thousands of entities">

### Don't flag

Deliberate choices a pass should not report as problems.

- <choice and why it is deliberate>

---

## Code standards

What clean code means in this project. Every skill that writes code follows it, and
`/analyse-code` and `/review-change` measure against it. Conventions already documented in
`CLAUDE.md` don't need repeating.

<!-- Short rules, project-specific. Examples: layering ("UI never touches the save format
directly"), size limits, error-handling style, naming, what gets tests. -->
- <rule>
- Tests: new behaviour and bug fixes get tests where the project has a test suite.

---

## Hands-off paths

Never edited by any skill (vendored code, scratch files, assets you manage yourself). Not
reported as findings either.

- `<path>`: <why>

---

## Ask before

Changes the skills must not make without your OK (runs ask about such items before they
start, all at once). These add to the generic rules: big refactors, items marked "verify",
design questions.

- <kind of change>

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

<!-- One line each: the scope, then what it covers. -->
- `<scope>`: <area>
