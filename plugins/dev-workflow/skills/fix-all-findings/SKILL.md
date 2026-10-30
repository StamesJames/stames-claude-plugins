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
  `Investigate and fix if confirmed` (recommended) and `Skip it`.
- **Order** for the triage: the user's findings first (a `CLAUDE_ANALYSIS.md` item only goes
  before them with a higher severity), each file by its section 4, then severity.

Arguments (all optional): a maximum number of fixes (`5`), `skip=<ids>`, and `user-only`: then
only `USER_ANALYSIS.md` is worked off. Count and triage only its items, and pass `user-only`
to every subagent (skill extras `user-only`). `CLAUDE_ANALYSIS.md` stays out of the run; the
final report may say in one line how many items are open there.
