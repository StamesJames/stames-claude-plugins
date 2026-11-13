# Plan

The concrete features to build, written by `/plan-features` from your ideas in
`USER_IDEAS.md` and your answers. An entry here is approved for building:
`/implement-next-feature` builds one, `/implement-all-features` works off the whole list.
You may edit entries or add your own; **Notes for Claude:** are binding. Rules (ids, stamps,
deleting done items) are in `list-format.md` of the `dev-workflow` plugin's `setup-workflow`
skill. All paths are relative to the repo root unless noted. Items are roughly ordered by
priority.

**[HOLD]** after the severity means: don't work on this item until the tag is removed.
**[CONFLICT]** marks a feature that contradicts the design docs; its **Decisions:** line
records that you want it anyway.

Passes:

- 2026-11-13 16:19 CET at b21c883: 1 wish (tiered verification) planned into 2 features (F1, F2), 0 kept as ideas.

Legend: **[H]** most wanted, **[M]** wanted, **[L]** nice to have.

---

## Concrete features

_Next id: F3_

### F2 [M] Full verification once after a run
_Written: 2026-11-13 16:19 CET (`b21c883`)_

**Idea:** free-text wish (2026-11-13), second half: with tiered verification (F1) the slow
**Full** tier runs rarely per item, so a run checks everything once at its end.

**What:** after the last item of a run (and before the after-run analysis) the driver starts
one headless agent that runs the **Full** tier of `### Verification`. If it fails, the agent
records an [H] finding in `CLAUDE_ANALYSIS.md` with the failing output and the run's commits as
suspects and commits it; the run report names it. Nothing is reverted.

**Why:** the per-item tiers trade some safety for speed; one full check per run catches what
the lighter sets missed while the run's commits are fresh, and the finding feeds the next fix
run.

**Done when:**
- `USER_WORKFLOW.md` template → `## Runs` gains `Full verification after run: features yes,
  fixes yes` (with a comment); `workflow-lists` reads it (default yes for both when missing)
  and `print_settings` shows it.
- `run go` (in `after_run` or right before it, on `EMPTY` / `LIMIT`): when the setting is on
  for the run's kind, the run made commits and `### Verification` has a **Full** block, one
  headless agent (model: the analysis model, role `verify`) runs it; otherwise the log says why
  it was skipped.
- The agent's prompt: run only the Full tier; on success end with `RESULT: PASSED`; on failure
  add one [H] finding to `CLAUDE_ANALYSIS.md` with `workflow-lists add` (failing command, the
  relevant output, the run's item commits `start..HEAD` as suspects), commit it (`chore`, no
  `Refs:`), and end with `RESULT: FAILED <one line>`. It never changes project code.
- The run log and report carry the outcome (`full verification: passed` / `failed → finding
  <id>` / `skipped: <why>`); the stats line has role `verify`.
- The after-run analysis still runs after it and sees the new finding.

**Verify:**
- Scratch repo with a tiered `CLAUDE.md` and a mock `claude` (`WORKFLOW_LISTS_CLAUDE`): a run
  with a committing item and a passing verify mock reports `passed`; a failing mock that adds a
  finding and commits leaves that commit and finding, and the report names it; a run with no
  commits, and one without a **Full** block, report `skipped` and start no verify agent; the
  stats file has a `verify` row.
- The smoke test against fresh templates prints `OK` (new `## Runs` setting in the template).
- Validation passes; cross-reference grep prints only `done`.

**Design docs:** `CLAUDE.md` → "The `dev-workflow` plugin" (`bin/workflow-lists` driver, `##
Runs` settings read by the script); `loop.md` (what the run does unattended).

**How:** `bin/workflow-lists`: `DEFAULT_RUNS` / `parse_runs` for the setting, a
`full_verification(run)` step called from `run go` before `after_run`, its prompt, stats role
`verify`, report line; `templates/USER_WORKFLOW.md` `## Runs`; `loop.md` (the report lists it);
this repo's `USER_WORKFLOW.md` only if the user asks; `CLAUDE.md` driver description. Builds on the tiers F1 added.

**Size:** M. Risk: a flaky full verification creating noise findings; the finding names the
output so the user can judge.

**Decisions:** On failure: report it and add an [H] finding to `CLAUDE_ANALYSIS.md` (failing
output, the run's commits as suspects), committed; nothing reverted, no bisecting. Small
details chosen: setting default yes for both kinds; runs before the after-run analysis; skipped
without commits or without a Full tier; analysis model; stats role `verify` (2026-11-13)

**Model:** sonnet

## Suggested order

1. **F2**: builds on the **Full** tier F1 added.
