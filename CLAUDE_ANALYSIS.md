# Cleanup analysis

Running list of open cleanup findings, written by `/analyse-code` and worked off by
`/fix-next-finding` and `/fix-all-findings`. Rules (ids, stamps, deleting fixed items) are in
`list-format.md` of the `dev-workflow` plugin's `setup-workflow` skill. All paths are relative to the repo root
unless noted. Items are roughly ordered by priority inside each section.

Passes:

- 2026-11-01 08:36 CET at 78d25f9: full (whole code scope, read directly: 30 files, ~3000 lines), 11 new, 0 fixed.

Legend: **[H]** high — fix before building more on top, **[M]** medium, **[L]** low / nice to have.

---

## 1. Bugs

_Next id: 1.6_

### 1.2 [M] Decision step: duplicate skip options, and `Skip it` has no recording rule
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`plugins/dev-workflow/skills/fix-all-findings/SKILL.md:15-17`,
`plugins/dev-workflow/skills/setup-workflow/loop.md:73-80`, `loop.md:81-89`

For a "verify" finding, `fix-all-findings` tells triage to offer `Investigate and fix if
confirmed` and `Skip it`; `loop.md` 2.2 then adds `Skip this run` to every item's first
question, so the user sees two skip options. `loop.md` 2.3 has no rule for `Skip it`, so it
falls under "Otherwise add **Decisions:**": the entry gets `**Decisions:** Skip it` and the
item is queued as decided, leaving the item agent to interpret it. Separately, a `_Failed:_`
item already has four options (the `AskUserQuestion` maximum), so adding `Skip this run`
drops "the weakest", which is undefined; an unattended orchestrator may drop `Drop it` or
`Put on hold` arbitrarily.

**Fix:** remove `Skip it` from `fix-all-findings` (`Skip this run` covers it). In `loop.md`
2.2, name the option that gives way for `_Failed:_` items (e.g. `Drop it`, still reachable
through "Other"), or fold `Skip this run` into the failed-item option set explicitly.

**Decisions:** For `_Failed:_` items, `Drop it` gives way to `Skip this run` (still reachable through "Other"); remove `Skip it` from `fix-all-findings`. (2026-11-01)

### 1.3 [M] `loop.md` hands literal `${CLAUDE_PLUGIN_ROOT}` paths to subagents
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`plugins/dev-workflow/skills/setup-workflow/loop.md:55-56`, `loop.md:107`, `loop.md:135`,
`loop.md:184`, `plugins/dev-workflow/skills/plan-features/SKILL.md:57-62`

Claude Code substitutes `${CLAUDE_PLUGIN_ROOT}` only in loaded skill content; `loop.md` is
read with the Read tool, so its prompts contain the literal placeholder, and the variable is
not set in the Bash tool's environment (`echo $CLAUDE_PLUGIN_ROOT` is empty in a session with
the plugin enabled). The triage `Plan` subagent must read `list-format.md` and `work-item.md`
by these paths; it only works if the orchestrator notices and substitutes them, which nothing
tells it to do (possible: a capable orchestrator often does). The fallback paths in steps 3, 4
and 7 have the same problem. `plan-features`' per-idea `Plan` subagents are told to read
`list-format.md` with no path at all.

**Fix:** add one line before the prompts in `loop.md`: replace `${CLAUDE_PLUGIN_ROOT}` with
the plugin root (two directories above this file) before sending a prompt. In
`plan-features`, give the subagents the full `${CLAUDE_PLUGIN_ROOT}/skills/setup-workflow/list-format.md`
path (substituted there, since it is a SKILL.md).

**Decisions:** Investigate and fix if confirmed (triage confirmed `$CLAUDE_PLUGIN_ROOT` is empty in Bash). (2026-11-01)

### 1.4 [L] No defined result when the given id's entry no longer exists
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`plugins/dev-workflow/skills/setup-workflow/work-item.md:138`,
`plugins/dev-workflow/skills/setup-workflow/loop.md:120`,
`plugins/dev-workflow/skills/fix-next-finding/SKILL.md:35-37`,
`plugins/dev-workflow/skills/implement-next-feature/SKILL.md:30-32`

`work-item.md` H allows `RESULT: NOTHING_LEFT` "only when picking yourself", but `loop.md`
3.4 reads `NOTHING_LEFT` as "the item was gone". Scenario: fixing 1.2 also resolves 3.1 and
deletes both entries (`fix-next-finding` says to); 3.1 is still queued, and its agent, given
`3.1`, finds no entry and has no result line that fits.

**Fix:** in `work-item.md` H (and section 1 of both single-item skills), say that a given id
whose entry no longer exists ends with `RESULT: NOTHING_LEFT`, matching `loop.md`.

**Model:** haiku

### 1.5 [L] Version numbers in a suggested-order section are flagged as dangling ids
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`plugins/dev-workflow/bin/workflow-lists:82`, `bin/workflow-lists:479-484`

`ANALYSIS_REF_RE` matches any `<n>.<n>` in section 4 of the analysis files, which is free
prose. A line like "after the 3.0 migration, **1.2** first" gives `error … suggested order
names 3.0, which is not an open item`, and the hook reports it after every edit of the list,
pushing an agent to reword correct text.

**Fix:** in suggested-order sections only count ids that are bold (`**1.2**`, the format
`list-format.md` shows) or that are followed by a non-digit separator and match an id pattern
of an existing section; simplest: require the bold form in section 4 and document that in
`list-format.md`. Run the smoke test.

---

## 2. Performance

_Next id: 2.1_

_No open items._

---

## 3. Cleanliness / architecture

_Next id: 3.7_

### 3.1 [L] `SCOPE:` lines of the review and after-run analysis agents are not collected
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`plugins/dev-workflow/skills/setup-workflow/loop.md:113-114`, `loop.md:138-140`,
`loop.md:179-185`, `loop.md:198`

`loop.md` 3.4 collects `SCOPE:` lines only from item agents, but `review-change` and
`analyse-code` print one too when their commit has no scope. Sections 4 and 7 say nothing
about them, so the final report's "collected `SCOPE:` lines" misses them.

**Fix:** in sections 4 and 7, collect `SCOPE:` lines like in 3.4.

**Model:** haiku

### 3.2 [L] Reasoning levels differ between `list-format.md` and the script
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`plugins/dev-workflow/skills/setup-workflow/list-format.md:88-91`,
`plugins/dev-workflow/bin/workflow-lists:52`

`list-format.md` lists the levels `low`, `medium`, `high`, `xhigh`; `EFFORTS` in the script
also accepts `max`. A rule and its check should agree.

**Fix:** pick one: add `max` to `list-format.md` (the Agent tool accepts it) or drop it from
`EFFORTS`.

**Model:** haiku

**Decisions:** No `max`: remove it from `EFFORTS` in `bin/workflow-lists`. (2026-11-01)

### 3.3 [L] `implement-all-features` repeats a final-report rule from `loop.md`
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`plugins/dev-workflow/skills/implement-all-features/SKILL.md:26-27`,
`plugins/dev-workflow/skills/setup-workflow/loop.md:195`

"The final report also lists the `DEFAULTS:` … and what couldn't be verified" is already in
`loop.md` section 8 for every run; the code standards say a rule lives in one place.

**Fix:** delete the paragraph.

**Model:** haiku

### 3.4 [L] `analyse-code` is ambiguous about splitting a small `full` scope
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`plugins/dev-workflow/skills/analyse-code/SKILL.md:3`, `analyse-code/SKILL.md:52-56`

The description says `full` analyses "with one subagent per area"; the body says "For `full`
(or any scope too large …, roughly more than 40 files or 8000 lines)", which reads both as
"always split `full`" and as "split `full` only above the threshold". In a small repo like
this one (30 files) a pass has to guess.

**Fix:** state the rule once, e.g. "Split when the scope is above roughly 40 files or 8000
lines (usual for `full`); smaller scopes are read directly", and match the description.

**Model:** haiku

### 3.5 [L] Long unwrapped prose lines in rule files and templates
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`plugins/dev-workflow/skills/setup-workflow/work-item.md:3`,
`plugins/dev-workflow/skills/setup-workflow/SKILL.md:57`, `setup-workflow/SKILL.md:101`,
`plugins/dev-workflow/skills/setup-workflow/list-format.md:72`,
`plugins/dev-workflow/skills/setup-workflow/templates/CLAUDE_ANALYSIS.md:5`,
`templates/USER_ANALYSIS.md:5-6`, `templates/claude-md-section.md:5`, `:7`, `:9`,
`plugins/agent-context/skills/tidy-claude-md/SKILL.md:64-66`

These lines run to 102–151 characters (apparently left from in-place path replacements), or
break mid-sentence (`tidy-claude-md` "belong to" / "something else"), against the code
standard of wrapping like the surrounding file (~95). The templates copy the long lines into
every target project (this repo's own `CLAUDE_ANALYSIS.md`, `USER_ANALYSIS.md` and
`CLAUDE.md` → `## Workflow` have them too).

**Fix:** re-wrap these paragraphs to ~95 characters; table rows and frontmatter descriptions
stay as they are. Run the smoke test for the template changes.

**Model:** haiku

### 3.6 [L] Root README's "Adding a plugin" steps miss the table row
_Found: 2026-11-01 08:36 CET (`78d25f9`)_

`README.md:20-24`

`CLAUDE.md` says adding a plugin includes adding a row to the table in `README.md`; the
README's own numbered steps don't mention it, so following them leaves the plugin unlisted.

**Fix:** add the step ("Add a row to the table above") between steps 2 and 3.

**Model:** haiku

---

## 4. Suggested order of work

1. **1.2**: in the decision path of `loop.md` and `fix-all-findings`.
2. **1.3**: one line in `loop.md` plus a path in `plan-features`.
3. **1.4** together with **3.1** and **3.3**: small wording fixes in `loop.md`,
   `work-item.md` and the loop skills.
4. **1.5** together with **3.2**: rules vs. script; both need the smoke test.
5. **3.4**.
6. **3.5** together with **3.6**: wrapping and README, docs only.
