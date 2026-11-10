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

The loops ask every needed decision up front, then run unattended: failed attempts are saved
on `workflow/failed/<item>` branches, larger items get a fresh review, and feature runs end
with an analysis of what changed (split across agents when the run was large). Models per kind of work are set in the project's
`USER_WORKFLOW.md` (`## Runs`) and can be overridden per item.

Everything mechanical is done by the bundled `workflow-lists` command (Python 3, on the Bash
tool's `PATH` while the plugin is enabled), so agents spend tokens on the work, not on
bookkeeping: it checks the lists (`check`), prints only what an agent needs (`summary`,
`show`, `find`, `section`), edits entries (`add`, `delete`, `set`, `move`, `pass`: ids, stamps,
mentions, failed branches), orders and pre-triages a run (`triage`) and holds a run's state
(`run`: queue, escalation, failures, review decision, report), so long runs survive
interruptions and context compaction. A `PostToolUse` hook runs the check after every hand
edit of a list file and reports only errors. Because of the `bin/` directory the plugin is
for Claude Code only (Cowork and claude.ai don't install it).

Run them as `/dev-workflow:<skill>` (or just `/<skill>` when the name is unambiguous).
Start in a project with `/dev-workflow:setup-workflow`.

Install it from the `stames-plugins` marketplace; see the [repo README](../../README.md).
