# Card 4 — prior-plan-stencil

**Create:**
- `contracts/stencils/loom/loom-template-prior-plan.md`
- `plan:contracts/stencils#LoomTemplatePriorPlan` -> `var LoomTemplatePriorPlan []byte`
- `contracts/stencils/priorplantemplate_test.go`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/plantemplate_test.go`

**Intent:**
Add the stencil holding the prior-plan block's text, and tell the plan template to read it (decision record D2, D5).

`loom-template-prior-plan.md` opens with the leading HTML comment banner every loom stencil carries (what reads it, and that it is appended to the end of the Plan prompt rather than rendered through a marker), followed by a `## Prior plan` section.
It has exactly two top-level markers, both required: `{{.archive_dir}}`, the archive directory's absolute path, and `{{.moved_files}}`, the list of moved file names.
No `{{if}}`/`{{range}}` conditionals.
The section tells the session that an earlier session of this run wrote a plan, which was moved into the archive directory, and instructs it to:

- read the archived plan before writing anything;
- check it against the decision record and the current tree;
- carry forward every card that still holds by copying it into the plan directory rather than re-deriving it;
- re-verify every glyph it carries forward with `lyx quarry`;
- revise only what is wrong, missing or stale;
- never modify the archive directory;
- never copy `00-overview.md` from the archive, but write it fresh, last, with `approved: false`.

Write it with semantic line breaks, and cite no loomyard-internal path (the citation enforcement scan in this package applies).

In `stencils.go`, add the `//go:embed loom/loom-template-prior-plan.md` var `LoomTemplatePriorPlan` beside the other loom stencils, and register `{"loom-template-prior-plan", &LoomTemplatePriorPlan}` in `entries` directly after `loom-rubric-webster-review`, so the registry order before it (which `internal/stencilcli`'s integration test indexes by position) is unchanged.

In `loom-template-plan.md`'s Step 3, add one static line telling the agent that when the prompt ends with a `Prior plan` section, it must read that section and act on it before writing anything in this step.
The template stays conditional-free.

Tests:

- `priorplantemplate_test.go`: `stencil.TopLevelMarkers(LoomTemplatePriorPlan)` is exactly `archive_dir` and `moved_files`; the text carries a short distinctive phrase for each instruction above (read before writing, carry forward by copying, re-verify with `lyx quarry`, never modify the archive, never copy `00-overview.md`, `approved: false`), following `plantemplate_test.go`'s substring precedent.
- `plantemplate_test.go`: add a row asserting the template names the trailing `Prior plan` section.
- The registry-versus-disk test in `registry_test.go` covers the new file without change.

**ImpactSummary:** one new registry row appended after the loom rubrics and one added template line; `lyx stencil list` and seeding pick the stencil up through the registry, and no existing stencil's markers change.
