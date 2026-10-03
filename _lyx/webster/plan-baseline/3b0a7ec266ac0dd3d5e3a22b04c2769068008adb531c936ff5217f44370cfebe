# Card 9 — fabricengine-hub-fixture

**Edit:**
- `fabricengine/add_branch_exists_test.go`
- `fabricengine/add_remoteleftover_integration_test.go`
- `fabricengine/add_rollback_adopt_test.go`
- `fabricengine/archive_integration_test.go`
- `fabricengine/checkout_index_refresh_test.go`
- `fabricengine/checkout_rollback_test.go`
- `fabricengine/cleanup_primary_integration_test.go`
- `fabricengine/cleanupremote_integration_test.go`
- `fabricengine/committedfile_integration_test.go`
- `fabricengine/destroy_containment_toctou_integration_test.go`
- `fabricengine/destructivegaps_integration_test.go`
- `fabricengine/headsha_integration_test.go`
- `fabricengine/healthreason_integration_test.go`
- `fabricengine/hubcontainment_integration_test.go`
- `fabricengine/hubreservedroutes_integration_test.go`
- `fabricengine/junction_pattern_integration_test.go`
- `fabricengine/launchers_containment_integration_test.go`
- `fabricengine/mutation_record_integration_test.go`
- `fabricengine/pairbranch_archive_integration_test.go`
- `fabricengine/portals_containment_integration_test.go`
- `fabricengine/prune_dirty_integration_test.go`
- `fabricengine/prune_unowned_integration_test.go`
- `fabricengine/reconcile_empty_anchor_integration_test.go`
- `fabricengine/reconcile_stale_registration_test.go`
- `fabricengine/reconcile_stale_removal_test.go`
- `fabricengine/remove_archive_integration_test.go`
- `fabricengine/remove_guard_integration_test.go`
- `fabricengine/remove_junctions_integration_test.go`
- `fabricengine/remove_refusal_remedy_integration_test.go`
- `fabricengine/remove_reserved_integration_test.go`
- `fabricengine/remove_siblingdirty_integration_test.go`
- `fabricengine/remove_warpbranch_integration_test.go`
- `fabricengine/removeremote_integration_test.go`
- `fabricengine/status_pollution_integration_test.go`
- `fabricengine/unwire_test.go`
- `fabricengine/warpforward_integration_test.go`
- `fabricengine/doc.go`

**Uses:**
- `internal/hubforge#Hub`

**Intent:**
Delete the `fabricFixture` shim so fabricengine tests speak `hubforge` directly (D5, #325 §2).

Delete `fabricFixture` and `newFabricFixture` from `reconcile_stale_registration_test.go`.
Each caller builds `h := hubforge.NewHub(t, ".")` and reads the hub's own fields: `Container` → `h.Container`, `Hub` → `h.PrimeWorktree()`, `Bare` → `h.WarpBare`, `WeftPrime` → `h.PrimeWeft()`, `WeftBare` → `h.WeftBare`, `Layout` → `h.Location`.
Where a test then builds `fabricengine.NewTopology(fabricengine.Config{})` by hand, it uses `h.Topology` instead.
An untagged test with no hub (such as `add_test.go`) keeps its hand-built topology, since building a hub there would break Tier Purity.
No file introduces a replacement wrapper type: that is the D6 hubforge clause card 8 records.

`fabricengine/doc.go` gains the #325 rules going forward: hub fixtures come only from `hubforge`, test git plumbing only from `gitkit`, and a new test file prefers the external `fabricengine_test` package plus `export_test.go` accessors, an internal-package test file needing a stated reason.

The `livestate_*` harness is not in this card's files and is not touched.
Assertions are unchanged; apply D10 to every touched test.

**ImpactSummary:** Integration-tagged fabricengine tests switch from the local fixture view to *hubforge.Hub and h.Topology; one package doc gains the test-layout rules.
