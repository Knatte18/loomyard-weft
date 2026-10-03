# Card 10 — fabricengine-pair-and-open-sites

**Edit:**
- `fabricengine/add_remoteleftover_integration_test.go`
- `fabricengine/add_rollback_adopt_test.go`
- `fabricengine/add_runrecords_integration_test.go`
- `fabricengine/checkout_index_refresh_test.go`
- `fabricengine/checkout_rollback_test.go`
- `fabricengine/destroy_containment_toctou_integration_test.go`
- `fabricengine/destructivegaps_integration_test.go`
- `fabricengine/hubcontainment_integration_test.go`
- `fabricengine/junction_pattern_integration_test.go`
- `fabricengine/launchers_containment_integration_test.go`
- `fabricengine/livestate_verbs_test.go`
- `fabricengine/merge_target_integration_test.go`
- `fabricengine/mergein_integration_test.go`
- `fabricengine/mergein_recovery_integration_test.go`
- `fabricengine/mergesiblings_integration_test.go`
- `fabricengine/mutation_record_integration_test.go`
- `fabricengine/origin_integration_test.go`
- `fabricengine/portals_containment_integration_test.go`
- `fabricengine/prune_dirty_integration_test.go`
- `fabricengine/prune_unowned_integration_test.go`
- `fabricengine/reconcile_stale_registration_test.go`
- `fabricengine/reconcile_stale_removal_test.go`
- `fabricengine/remove_archive_integration_test.go`
- `fabricengine/remove_junctions_integration_test.go`
- `fabricengine/remove_refusal_remedy_integration_test.go`
- `fabricengine/remove_siblingdirty_integration_test.go`
- `fabricengine/removeremote_integration_test.go`
- `fabricengine/unwire_test.go`
- `fabricengine/warpbinding_reconcile_integration_test.go`

**Uses:**
- `internal/hubforge#AddPairWith`
- `internal/hubforge#OpenFabric`

**Intent:**
Route fabricengine's repeated pair-creation and fabric-open setup through the hubforge builders card 8 added (#325 §2).

Each four-line `if _, err := topology.Add(l, slug, AddOptions{SkipPush: true}); err != nil { t.Fatalf(...) }` setup over a hub becomes `hubforge.AddPairWith(t, h, slug, fabricengine.AddOptions{SkipPush: true})`, keeping its `AddResult` where the test reads it.
Each `lyxcwd.ResolveWorktree(h.PrimeWorktree())` followed by `fabricengine.Open(l)` becomes `hubforge.OpenFabric(t, h)`, keeping the resolved `*lyxcwd.Location` from `h.Location` where the test still needs it.
A call whose `Add` result or error is itself the behaviour under test (an expected refusal, a rollback) stays a direct `h.Topology.Add` call: only setup converts.

The `livestate_*` harness is D9-protected: `livestate_verbs_test.go` converts its setup only, every assertion survives.
Apply D10 to every touched test.

**ImpactSummary:** Integration-tagged fabricengine setup code only; repeated Add/Open sequences collapse onto two hubforge calls, assertions unchanged.
