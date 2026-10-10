# Card 17 — reclassify-fabricengine-add-to-dot

**Edit:**
- `internal/fabricengine/add_branch_exists_test.go`
- `internal/fabricengine/add_pushretry_integration_test.go`
- `internal/fabricengine/add_remoteleftover_integration_test.go`
- `internal/fabricengine/add_rollback_adopt_test.go`
- `internal/fabricengine/add_rollback_branch_integration_test.go`
- `internal/fabricengine/add_runrecords_integration_test.go`
- `internal/fabricengine/archive_integration_test.go`
- `internal/fabricengine/checkout_index_refresh_test.go`
- `internal/fabricengine/checkout_rollback_test.go`
- `internal/fabricengine/cleanreason_integration_test.go`
- `internal/fabricengine/cleanup_primary_integration_test.go`
- `internal/fabricengine/cleanupremote_integration_test.go`
- `internal/fabricengine/cleanupremotewarp_integration_test.go`
- `internal/fabricengine/coalesce_integration_test.go`
- `internal/fabricengine/commit_integration_test.go`
- `internal/fabricengine/committedfile_integration_test.go`
- `internal/fabricengine/configchanges_integration_test.go`
- `internal/fabricengine/config_driven_junctions_integration_test.go`
- `internal/fabricengine/destroy_containment_toctou_integration_test.go`
- `internal/fabricengine/destroyremote_integration_test.go`
- `internal/fabricengine/destructivegaps_integration_test.go`
- `internal/fabricengine/diff_integration_test.go`
- `internal/fabricengine/dotlyxjunction_integration_test.go`

**Uses:**
- `internal/hubforge#CopyHub`
- `internal/hubforge#SharedHub`
- `internal/hubforge#Shape`

**Intent:** The first third of fabricengine's hub-taking tests move onto the template model under the classification in the overview's Shared Decisions.
Every test in these files runs a fabric verb, seeds config, adds a pair or spawns git against its hub, so every one is a write and takes `CopyHub` of the nearest shape: the anchor its `NewHub` call names, the one-pair shape where the test's first act is `AddPair` of a slug it reads back only through the `Hub` accessors, and the no-pair shape otherwise, keeping its own `AddPair`.
`configchanges_integration_test.go` iterates anchors from its table; each case takes the copy of its anchor's shape.
Pure refactor of fixtures: the existing assertions cover it.

**ImpactSummary:** Fixture construction changes in twenty-three test files; no production code and no assertion changes.
