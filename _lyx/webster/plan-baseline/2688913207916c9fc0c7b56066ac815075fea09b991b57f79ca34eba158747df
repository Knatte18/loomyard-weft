# Card 20 — reclassify-fabriccli-and-geometry

**Edit:**
- `internal/fabriccli/adoptremote_integration_test.go`
- `internal/fabriccli/cleanhub_integration_test.go`
- `internal/fabriccli/cleanupremove_integration_test.go`
- `internal/fabriccli/hubmutation_integration_test.go`
- `internal/fabriccli/mergescenario_integration_test.go`
- `internal/fabriccli/weftskippush_integration_test.go`
- `internal/hubreconcile/ensure_integration_test.go`
- `internal/hubgeom/geometry_integration_test.go`
- `internal/preflight/preflight_integration_test.go`
- `internal/preflightshed/preflight_integration_test.go`
- `internal/pairteardown/teardown_integration_test.go`
- `internal/gatecli/test_integration_test.go`
- `internal/configcli/configcli_integration_test.go`
- `internal/boardengine/boardtest/sync_test.go`
- `cmd/lyx/stencilseed_integration_test.go`
- `internal/idecli/cli_test.go`
- `internal/ideengine/spawn_scenario_integration_test.go`
- `internal/stencilcli/cli_integration_test.go`

**Uses:**
- `internal/hubforge#CopyHub`
- `internal/hubforge#SharedHub`
- `internal/hubforge#Shape`

**Intent:** The hub-taking tests of fabriccli and the geometry, preflight, config, board, ide and stencil packages, under the classification in Shared Decisions.
`hubgeom`'s geometry test reads geometry in its first steps and adds a pair in a later one, so it copies the `.` no-pair shape; a reviewer splitting the reads into a `SharedHub` test is a later economy, not this card's.
`ideengine`'s `TestSpawnAnchoredScenario` anchors at `wts/some-task`, outside the set, and stays on `NewHub`; its sibling scenario at `.` copies.
`gatecli`'s scenario seeds `gate.yaml` and so copies.
Every other test runs a verb or the binary and copies the shape its call names.
Pure refactor of fixtures: the existing assertions cover it.

**ImpactSummary:** Fixture construction changes in eighteen test files across twelve packages; no production code and no assertion changes.
