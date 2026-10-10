# Card 18 — reclassify-fabricengine-head-to-push

**Edit:**
- `internal/fabricengine/headsha_integration_test.go`
- `internal/fabricengine/healthreason_integration_test.go`
- `internal/fabricengine/hook_test.go`
- `internal/fabricengine/housekeeping_integration_test.go`
- `internal/fabricengine/hubcontainment_integration_test.go`
- `internal/fabricengine/index_integration_test.go`
- `internal/fabricengine/junction_pattern_integration_test.go`
- `internal/fabricengine/junction_repoint_test.go`
- `internal/fabricengine/launchers_containment_integration_test.go`
- `internal/fabricengine/livestate_manifest_selftest_test.go`
- `internal/fabricengine/livestate_matrix_test.go`
- `internal/fabricengine/livestate_refusal_selftest_test.go`
- `internal/fabricengine/livestate_states_selftest_test.go`
- `internal/fabricengine/livestate_verbs_selftest_test.go`
- `internal/fabricengine/mergecrucible_integration_test.go`
- `internal/fabricengine/mergein_integration_test.go`
- `internal/fabricengine/mergesiblings_integration_test.go`
- `internal/fabricengine/mergestateactive_integration_test.go`
- `internal/fabricengine/mergestate_integration_test.go`
- `internal/fabricengine/mutation_record_integration_test.go`
- `internal/fabricengine/openparent_integration_test.go`
- `internal/fabricengine/origin_integration_test.go`
- `internal/fabricengine/pairbranch_archive_integration_test.go`
- `internal/fabricengine/pathspec_tolerance_coupling_integration_test.go`
- `internal/fabricengine/portals_containment_integration_test.go`
- `internal/fabricengine/prune_dirty_integration_test.go`
- `internal/fabricengine/prune_unowned_integration_test.go`
- `internal/fabricengine/pull_integration_test.go`
- `internal/fabricengine/pushanchored_integration_test.go`

**Uses:**
- `plan:internal/hubforge#CopyHub`
- `plan:internal/hubforge#SharedHub`
- `plan:internal/hubforge#Shape`

**Intent:** The second third of fabricengine's hub-taking tests, under the same classification as card 17.
The live-state matrix, its self-tests, `openparent_integration_test.go` and `mergein_integration_test.go` iterate the anchors `.` and `backend`, both in the shape set, so every cell takes the copy of its anchor's shape; every cell arranges a state or runs a verb, so none is read-only.
`headsha_integration_test.go` and `index_integration_test.go` open a `Fabric` and then commit or write the index, so they copy too.
Pure refactor of fixtures: the existing assertions cover it.

**ImpactSummary:** Fixture construction changes in twenty-nine test files; no production code and no assertion changes.
