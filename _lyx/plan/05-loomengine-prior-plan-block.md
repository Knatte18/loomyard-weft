# Card 5 — loomengine-prior-plan-block

**Create:**
- `internal/loomengine/priorplan.go`
- `internal/loomengine#PriorPlanBlock`
- `internal/loomengine/priorplan_test.go`

**Uses:**
- `contracts/stencils#LoomTemplatePriorPlan`

**Intent:**
Add the renderer for the prior-plan block, in `loomengine` beside `composePlanPrompt`, which already reads the loom stencils through `stencilstore` (decision record D5).
Placing it here keeps `internal/loomshed`'s import allowlist unchanged: `loomshed` already imports `loomengine`.

`PriorPlanBlock` reads the `loom-template-prior-plan` stencil from `stencilsDir` with `stencilstore.Read` and fills it with `stencil.Fill`: `archive_dir` gets `archiveDir`, and `moved_files` gets `movedFiles` formatted as a markdown bullet list, one backtick-wrapped file name per line.
Both markers are required, so an empty `archiveDir` or an empty `movedFiles` fails with `stencil.Fill`'s unfilled-marker error rather than rendering a blank.
It returns the rendered text as a string; it performs no filesystem write and derives no path.

Tests in `priorplan_test.go`, seeding the embedded `stencils.LoomTemplatePriorPlan` into a temp stencils directory the way `prompt_test.go` seeds the plan template:

- The rendered block contains the `Prior plan` heading, the archive directory's absolute path, and every moved file name as its own bullet.
- An empty `archiveDir` returns an error naming `archive_dir`; an empty `movedFiles` returns an error naming `moved_files`.
- A stencils directory without the stencil returns an error.
