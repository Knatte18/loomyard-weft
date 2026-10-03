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
- `internal/loomcli/smoke_parentreview_test.go`
- `internal/shedadapters/bouncer_seed_test.go`
- `internal/shedadapters/bouncer_fixture_test.go`
- `internal/friction/friction_test.go`
- `internal/pattern/pattern_test.go`

**Uses:**
- `internal/testkit/stencilkit/stencilkit.go`

**Intent:** Convert the remaining hand-listed stencil seeders to `stencilkit` (D11), in the same shape as card 21: `loomengine`'s `newTestStencilsDir`, `newMinimalStencilsDir` (a deliberate-absence fixture, now "seed all, then remove X"), `newReworkStencilsDir`, `newPriorPlanStencilsDir` and `newParentReviewStencilsDir`;
`loomshed`'s `seedPriorPlanStencils`;
`loomrecipe`'s `seedBouncerStencils`;
`loomcli`'s `seedDiscussionStencil`, `seedPlanStencil`, the parent-review seeding in `parentreview_integration_test.go` and the hand-deployed parent-review stencils in the `smoke`-tagged `smoke_parentreview_test.go`;
`shedadapters`' `newBouncerFixture` default stencil map and its `writeBouncerStencils` writer in `bouncer_fixture_test.go`, with the per-test maps in `bouncer_seed_test.go` that feed it, now "seed all, then write the fixture rubric on top";
`friction`'s and `pattern`'s `newTestStencilsDir`.
Byte comparisons against an embedded default compare stamp-stripped bodies, modified-content fixtures write on top of a seed, and each converted test keeps its verdict.
After this card no test in the repo hand-lists stencil byte slices to seed a directory;
a grep of `_test.go` files for a `stencils.` identifier passed to `os.WriteFile` or held as a value in a name-to-content map is part of this card's own check.
`smoke_parentreview_test.go` is `smoke`-tagged, so this card runs `loomcli`'s smoke tier (D21);
a flaky result is re-run, not waved through.

**ImpactSummary:** Test fixtures only, across the loom packages, `shedadapters`, `friction` and `pattern`; every converted test keeps its assertions.

**Verify:** go test -tags smoke ./internal/loomcli/
