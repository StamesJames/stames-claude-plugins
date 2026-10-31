# dev-workflow

Claude Code plugin for a plan-first workflow built on running list files in your project.
Long runs open one fresh agent per item, so the context stays small and each item starts
clean.

- **Features:** `explore-feature` (work out a feature with Claude, write it to
  `USER_PLAN.md`), `plan-features` (Claude's own suggestions in `CLAUDE_PLAN.md`),
  `pitch-feature` (one suggestion: build now, approve for later, or drop),
  `implement-next-feature`, `implement-all-features`.
- **Code quality:** `analyse-code` (incremental by default, `full` splits across agents),
  `fix-next-finding`, `fix-all-findings` (`user-only` for your own findings),
  `review-change` (fresh-eyes cleanup of a recent change).
- **Setup:** `setup-workflow`.

The loops ask every needed decision up front, then run unattended: failed attempts are saved
on `workflow/failed/<item>` branches, larger items get a fresh review, and feature runs end
with an analysis of what changed. Models per kind of work are set in the project's
`USER_WORKFLOW.md` (`## Runs`) and can be overridden per item.

The list rules are checked mechanically by the bundled `workflow-lists` command (Python 3,
on the Bash tool's `PATH` while the plugin is enabled): `check` validates the lists, `summary`
prints one line per open item, `stamp` prints the stamp value. A `PostToolUse` hook runs the
check after every edit to a list file and reports only errors. Because of the `bin/`
directory the plugin is for Claude Code only (Cowork and claude.ai don't install it).

Run them as `/dev-workflow:<skill>` (or just `/<skill>` when the name is unambiguous).
Start in a project with `/dev-workflow:setup-workflow`.

Install it from the `stames-plugins` marketplace; see the [repo README](../../README.md).
