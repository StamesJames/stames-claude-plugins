# Analysing code: the checks, and one part of a split pass

Read by `analyse-code` (its checks apply to every pass) and by the read-only part agents of a
split pass, which `analyse-code` step 2 or `loop.md` step 7 starts. A part agent's prompt
names: the paths of its part, optionally a base commit (`since <commit>`), optionally
`after-run`, and optionally an output file.

## Part agents

Read-only: edit no project file and no list file.

1. Read `CLAUDE.md`, and of `USER_WORKFLOW.md` only `workflow-lists section USER_WORKFLOW.md
   "Analysis focus" "Code standards" "Hands-off"`. For `Model:`, the rubric is
   `workflow-lists section <this file's directory>/../setup-workflow/list-format.md
   "Choosing a model"`.
2. With a base commit: `git diff <base>..HEAD -- <paths>`; read the changed code and its
   surroundings, plus direct users of what changed where the change could break them (at
   most about ten files; if there are more, read a sample and say so in a `CROSS:` note).
   Without one: read the paths in full.
3. Apply the checks below. Skip generated code and the hands-off paths.
4. Return every finding as: `SECTION <1|2|3> | <H|M|L> | <title>`, then `paths:` with lines,
   two to four lines on what goes wrong (a concrete scenario) and why, `Fix:` the suggested
   change, `Model:` per the rubric, and `verify` if you could not confirm it. Then at most
   five lines of `CROSS:` notes about things that involve other parts. Nothing else.
5. If the prompt names an output file, write the findings there (create its directory; use a
   quoted heredoc, `cat > <file> <<'EOF'`) and answer only `PART DONE <n> findings`.
   Otherwise answer with the findings.

## The checks

Read the code itself, not only excerpts. Look for:

- **Bugs:** wrong behaviour, edge cases, persistence and load gaps, ordering of events or
  initialisation, failures only in release/export/production builds, error handling.
- **Performance:** work per frame/request/item that scales with data size, needless
  allocations, blocking I/O on hot paths, event storms.
- **Cleanliness/architecture:** violations of `## Code standards`, duplication, misplaced
  responsibilities, fragile coupling (load-bearing names, implicit ordering), naming clashes,
  speculative abstractions nothing uses, dead code or assets, stale comments, missing tests
  for important behaviour, and a stale `CLAUDE.md`.
- Everything listed under **Analysis focus** in `USER_WORKFLOW.md`, weighted as it says.

**`after-run`:** the changes come from a workflow run, where every item was self-reviewed
against the code standards and the larger ones got a fresh `/review-change`. Look for what
those per-item reviews can't see: bugs, performance, and problems that span several changes
(duplication between items, two items solving the same thing differently, fragile coupling
between them, a `CLAUDE.md` that no longer matches). Don't report naming, style or fit within
a single change.

Verify before recording. Check claims against the code and, where it helps, run the
verification from `CLAUDE.md` → `### Verification`. Mark anything you could not confirm as
"verify" or "possible" rather than stating it as fact. Do not record what
`USER_WORKFLOW.md` → **Don't flag** lists, nor conventions `CLAUDE.md` documents on purpose.
