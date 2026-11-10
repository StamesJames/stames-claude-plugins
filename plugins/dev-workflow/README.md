# dev-workflow

Claude Code plugin for a plan-first workflow built on running list files in your project.
Long runs open one fresh agent per item, so the context stays small and each item starts
clean.

- **Features:** you jot down ideas in `USER_IDEAS.md`; `plan-features` asks you the open
  questions and turns them into concrete features in `CLAUDE_PLAN.md`; `brainstorm-feature`
  talks a rough idea through with you first, then plans it straight away;
  `implement-next-feature` and `implement-all-features` build that plan. Claude's own ideas
  go to `CLAUDE_IDEAS.md` (`suggest-features`); `pitch-feature` pitches one, which you add to
  your ideas, plan right away, or drop.
- **Code quality:** `analyse-code` (incremental by default, `full` splits across agents),
  `fix-next-finding`, `fix-all-findings` (`user-only` for your own findings),
  `review-change` (fresh-eyes cleanup of a recent change).
- **Setup:** `setup-workflow`.

The loops ask every needed decision up front, then hand over to the bundled driver
(`workflow-lists run go`), which runs unattended in the background or from your shell: one
fresh headless agent (`claude -p`) per item with model, effort, turn and cost limits from
`USER_WORKFLOW.md` (`## Runs`), a fresh review that re-runs the verification and sends an
unmet item back for a rework round, failed attempts saved on `workflow/failed/<item>`
branches with one escalated retry, and a closing analysis of what changed (split across
parallel agents when the run was large). No session model spends tokens on the loop itself;
an interrupted run resumes where it stopped.

Every agent of a run writes one line to a stats file (plugin version, model, size, turns,
tokens, cost, outcome), and `workflow-lists stats` turns it into tables per plugin version,
model or kind, so a change to the plugin or the settings can be measured instead of guessed.

Everything mechanical is done by the bundled `workflow-lists` command (Python 3, on the Bash
tool's `PATH` while the plugin is enabled), so agents spend tokens on the work, not on
bookkeeping: it checks the lists (`check`), prints only what an agent needs (`brief`,
`summary`, `show`, `find`, `section`), edits entries (`add`, `delete`, `set`, `move`, `pass`:
ids, stamps, mentions, failed branches), orders and pre-triages a run (`triage`), holds a
run's state (`run`), drives it (`run go`) and reports on past runs (`stats`). A
`PostToolUse` hook runs the check after every hand edit of a list file and reports only
errors. Because of the `bin/` directory the plugin is for Claude Code only (Cowork and
claude.ai don't install it).

Run them as `/dev-workflow:<skill>` (or just `/<skill>` when the name is unambiguous).
Start in a project with `/dev-workflow:setup-workflow`.

Install it from the `stames-plugins` marketplace; see the [repo README](../../README.md).
