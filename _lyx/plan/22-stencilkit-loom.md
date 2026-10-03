# Card 22 — stencilkit-loom

**Edit:**
- `internal/loomengine/prompt_test.go`
- `internal/loomengine/rework_test.go`
- `internal/loomengine/priorplan_test.go`
- `internal/loomengine/parentreview_test.go`
- `internal/loomshed/planwrite_test.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/loomcli/parentreview_integration_test.go`
- `internal/shedadapters/bouncer_seed_test.go`
- `internal/friction/friction_test.go`
- `internal/pattern/pattern_test.go`

**Uses:**
- `internal/testkit/stencilkit/stencilkit.go`

**Intent:** Convert the remaining hand-listed stencil seeders to `stencilkit` (D11), in the same shape as card 21: `loomengine`'s `newTestStencilsDir`, `newMinimalStencilsDir` (a deliberate-absence fixture, now "seed all, then remove X"), `newReworkStencilsDir`, `newPriorPlanStencilsDir` and `newParentReviewStencilsDir`;
`loomshed`'s `seedPriorPlanStencils`;
`loomrecipe`'s `seedBouncerStencils`;
`loomcli`'s `seedDiscussionStencil`, `seedPlanStencil` and the parent-review seeding in `parentreview_integration_test.go`;
`shedadapters`' seeding in `bouncer_seed_test.go`;
`friction`'s and `pattern`'s `newTestStencilsDir`.
Byte comparisons against an embedded default compare stamp-stripped bodies, modified-content fixtures write on top of a seed, and each converted test keeps its verdict.
After this card no test in the repo hand-lists stencil byte slices to seed a directory;
a grep for `os.WriteFile` of a `stencils.` identifier in `_test.go` files is part of this card's own check.

**ImpactSummary:** Test fixtures only, across the loom packages, `shedadapters`, `friction` and `pattern`; every converted test keeps its assertions.
