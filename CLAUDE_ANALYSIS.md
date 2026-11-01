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

_No open items._

---

## 2. Performance

_Next id: 2.1_

_No open items._

---

## 3. Cleanliness / architecture

_Next id: 3.7_

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

1. **3.5** together with **3.6**: wrapping and README, docs only.
