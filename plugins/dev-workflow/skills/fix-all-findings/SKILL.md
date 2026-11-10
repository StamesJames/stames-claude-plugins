---
name: fix-all-findings
description: Work off CLAUDE_ANALYSIS.md and USER_ANALYSIS.md completely (with `user-only`, only USER_ANALYSIS.md): ask all needed decisions up front, then fix one finding per fresh subagent, unattended, with review and failure handling, until nothing fixable is left. Use when the user asks to fix all findings, work off the analysis list(s), or loop /fix-next-finding. Needs a project set up with /setup-workflow (USER_WORKFLOW.md).
---

# Fix all analysis findings

Follow `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/loop.md` exactly, with these specifics:

- **Kind of work:** `fixes` (models `fixes`; fresh review and analyse-after-run settings for
  fixes).
- **Lists:** `CLAUDE_ANALYSIS.md` and `USER_ANALYSIS.md`, sections 1–3. Ids of the user's items
  carry the `U` prefix (`U1.1`) everywhere outside the file.
- **Single-item skill:** `fix-next-finding`, description `Fix <id>`.
- **Triage cases** (needs the user, besides `work-item.md` B): findings marked "verify" or
  "possible", and findings asking about design intent. For a "verify" finding, offer
  `Investigate and fix if confirmed` (recommended); `Skip this run` (added by `loop.md`)
  covers skipping.
- **Triage depth:** `workflow-lists triage` marks every finding `CHECK`: look at the cited
  code to see whether the problem is still there and the fix is clear, no further. The
  user's short findings may cite nothing: then find the code just far enough to classify
  them.

Arguments (all optional): a maximum number of fixes (`5`), `skip=<ids>`, and `user-only`: then
only `USER_ANALYSIS.md` is worked off: `run start --user-only`, triage only that list, and
pass `user-only` to every item agent (skill extras `user-only`). `CLAUDE_ANALYSIS.md` stays out of the run; the
final report may say in one line how many items are open there.
