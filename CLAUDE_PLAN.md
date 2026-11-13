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

### F1 [H] Tiered verification for items and reviews
_Written: 2026-11-13 16:18 CET (`b21c883`)_

**Idea:** free-text wish (2026-11-13): slow verification (a Godot project's one big playthrough
script, 15–30 min per item, 13–22 min per review) runs at least twice per item; split it into
tiers and run only what the change needs.

**What:** a target project's `CLAUDE.md` → `### Verification` can be split into **Quick**
(every change), **Targeted** (per area: "touches `player/` → movement test") and **Full**
(slow; only in named cases, plus once after a run, F2). Item agents run quick, the targeted
checks for the areas they touched, and full only in its named cases or when the entry's
**Verify:** asks for it, and report what they ran. The review runs that same set once instead
of everything again.

**Why:** in projects with slow verification most of a run's wall-clock time is re-running
checks unrelated to the change (stats: reviews of 13–18 turns taking 13–22 min); tiers cut that
without weakening the acceptance gate.

**Done when:**
- `templates/claude-md-section.md` shows the tiered format (bold labels **Quick** /
  **Targeted** / **Full**, the Full label carrying its "when" cases), and says an untiered
  section counts as Quick (everything, every time: today's behaviour).
- `work-item.md` section E: run Quick, the Targeted checks whose areas the diff touches, Full
  only in its named cases or when **Verify:** names it; widen when unsure. Section H adds a
  `CHECKS: <id> — <tiers/checks run> (<why>)` line before `RESULT:`; `item-rules.md` carries
  the digest.
- `workflow-lists run`: `apply_result` accepts `CHECKS:` (RESULT_LINE_RE), logs it, shows it in
  the report, and the review prompt (`run go`) passes the item's CHECKS line to the reviewer;
  the rework prompt asks for the line too.
- `review-change` step 2 runs the item's set (from the CHECKS line, or by its own judgement
  without one) plus whatever the diff needs; a set that was clearly too light is a gap
  (`REWORK`, or with `final` run it and note it). Step 4 after its own edits reruns only Quick
  and the Targeted checks of the files it touched, never Full.
- `plan-features` step 2 / 4: **Verify:** names the tiers or checks a feature needs when the
  project's verification is tiered (e.g. "targeted: save/; full").
- `setup-workflow` B (Check) and A (setup): suggest splitting into tiers when the verification
  is untiered and slow (a step taking minutes) or when the run stats show long item/review wall
  times; never rewrite it without the user.
- `workflow-lists check` accepts both forms; a tiered section without a **Quick** block warns.

**Verify:**
- The scratch-repo smoke test prints `OK` with the new template, and again after replacing `###
  Verification` with an untiered one.
- Mock run (`WORKFLOW_LISTS_CLAUDE=<mock>`, `run start` / `run queue`, `run go features`): an
  item mock printing `CHECKS: F1 — quick, targeted save/ (touches save)` gets the line into the
  run log and report, and the review mock's prompt contains it.
- `claude plugin validate .` and `plugins/dev-workflow` pass (known README warning only); the
  cross-reference grep prints only `done`; `workflow-lists check` at the repo root prints `OK`.
- Read-through: no skill still says "every fix and feature runs these" / "run the verification"
  without the tier rule.

**Design docs:** `CLAUDE.md` → "The `dev-workflow` plugin" (shared rule files, `work-item.md`
section letters, result lines kept in sync with `workflow-lists run`);
`plugins/dev-workflow/README.md`.

**How:** prompt changes in `templates/claude-md-section.md`, `work-item.md` E and H,
`item-rules.md`, `review-change/SKILL.md` steps 2 and 4–5, `plan-features/SKILL.md` steps 2 and
4, `setup-workflow/SKILL.md` A and B; script: `RESULT_LINE_RE` and `apply_result` for
`CHECKS:`, the review and rework prompts in `run go`, the `check_claude_md` warning. Update
`CLAUDE.md` (result-line list) and the plugin README. Minor version bump. This repo's own `###
Verification` stays untiered.

**Size:** M (many files, each change small). Risk: agents choosing too light a set; the
reviewer's REWORK and F2's full run catch it.

**Decisions:** Tier format: bold labels **Quick** / **Targeted** / **Full** under `###
Verification`. Item agents report a new `CHECKS:` result line, which the driver logs and passes
to the reviewer. Review runs the item's set once (more if the diff needs it, REWORK if too
light); after its own edits only Quick plus affected Targeted, never Full. Untiered sections
behave as today (2026-11-13)

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
this repo's `USER_WORKFLOW.md` only if the user asks; `CLAUDE.md` driver description. Needs F1.

**Size:** M. Risk: a flaky full verification creating noise findings; the finding names the
output so the user can judge.

**Decisions:** On failure: report it and add an [H] finding to `CLAUDE_ANALYSIS.md` (failing
output, the run's commits as suspects), committed; nothing reverted, no bisecting. Small
details chosen: setting default yes for both kinds; runs before the after-run analysis; skipped
without commits or without a Full tier; analysis model; stats role `verify` (2026-11-13)

**Model:** sonnet

## Suggested order

1. **F1** + **F2**: F2 needs the **Full** tier F1 introduces.
