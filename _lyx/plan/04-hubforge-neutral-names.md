# Card 4 — hubforge's hub accessors take the code/records names

**Edit:**
- `//cmd/lyx/stencilseed_integration_test.go`
- `internal/battencli/lifecycle_integration_test.go`
- `internal/battencli/records_integration_test.go`
- `internal/boardengine/boardtest/sync_test.go`
- `internal/fabriccli/adoptremote_integration_test.go`
- `internal/fabriccli/cleanhub_integration_test.go`
- `internal/fabriccli/cleanupremove_integration_test.go`
- `internal/fabriccli/hubmutation_integration_test.go`
- `internal/fabriccli/mergescenario_integration_test.go`
- `internal/fabriccli/weftskippush_integration_test.go`
- `internal/fabricengine/add_remoteleftover_integration_test.go`
- `internal/fabricengine/add_rollback_adopt_test.go`
- `internal/fabricengine/archive_integration_test.go`
- `internal/fabricengine/checkout_rollback_test.go`
- `internal/fabricengine/cleanreason_integration_test.go`
- `internal/fabricengine/cleanup_primary_integration_test.go`
- `internal/fabricengine/cleanupremote_integration_test.go`
- `internal/fabricengine/cleanupremotewarp_integration_test.go`
- `internal/fabricengine/coalesce_integration_test.go`
- `internal/fabricengine/commit_integration_test.go`
- `internal/fabricengine/configchanges_integration_test.go`
- `internal/fabricengine/destroyremote_integration_test.go`
- `internal/fabricengine/destructivegaps_integration_test.go`
- `internal/fabricengine/diff_integration_test.go`
- `internal/fabricengine/hook_test.go`
- `internal/fabricengine/index_integration_test.go`
- `internal/fabricengine/livestate_manifest_selftest_test.go`
- `internal/fabricengine/livestate_refusal_selftest_test.go`
- `internal/fabricengine/livestate_states_selftest_test.go`
- `internal/fabricengine/livestate_verbs_test.go`
- `internal/fabricengine/mergecrucible_integration_test.go`
- `internal/fabricengine/mergein_integration_test.go`
- `internal/fabricengine/mergein_recovery_integration_test.go`
- `internal/fabricengine/mergelock_integration_test.go`
- `internal/fabricengine/mergesiblings_integration_test.go`
- `internal/fabricengine/mergestage_integration_test.go`
- `internal/fabricengine/mergestagetracked_integration_test.go`
- `internal/fabricengine/mergestateactive_integration_test.go`
- `internal/fabricengine/mergestate_integration_test.go`
- `internal/fabricengine/merge_target_integration_test.go`
- `internal/fabricengine/mergeweftlocal_integration_test.go`
- `internal/fabricengine/midmerge_integration_test.go`
- `internal/fabricengine/openparent_integration_test.go`
- `internal/fabricengine/pairbranch_archive_integration_test.go`
- `internal/fabricengine/pathspec_tolerance_coupling_integration_test.go`
- `internal/fabricengine/pull_integration_test.go`
- `internal/fabricengine/pushanchored_integration_test.go`
- `internal/fabricengine/reconcile_stale_registration_test.go`
- `internal/fabricengine/remove_archive_integration_test.go`
- `internal/fabricengine/remove_finish_integration_test.go`
- `internal/fabricengine/remove_guard_integration_test.go`
- `internal/fabricengine/remove_junctions_integration_test.go`
- `internal/fabricengine/remove_records_integration_test.go`
- `internal/fabricengine/remove_refusal_remedy_integration_test.go`
- `internal/fabricengine/removeremote_integration_test.go`
- `internal/fabricengine/remove_remotetask_integration_test.go`
- `internal/fabricengine/remove_siblingdirty_integration_test.go`
- `internal/fabricengine/resetpairwarp_integration_test.go`
- `internal/fabricengine/snapshot_integration_test.go`
- `internal/fabricengine/syncweft_integration_test.go`
- `internal/fabricengine/warpbinding_reconcile_integration_test.go`
- `internal/fabricengine/weftgit_exclude_test.go`
- `internal/fabricengine/weftgit_pathspec_integration_test.go`
- `internal/fabricengine/weftgit_unborn_warp_test.go`
- `internal/fabricengine/weftguards_integration_test.go`
- `internal/hubforge/doc.go`
- `internal/hubforge/hub.go`
- `internal/hubforge/hub_test.go`
- `internal/hubforge/seed.go`
- `internal/hubgeom/geometry_integration_test.go`
- `internal/landingshed/finalize_integration_test.go`
- `internal/loomcli/circling_integration_test.go`
- `internal/loomcli/integration_driverbootstrap_test.go`
- `internal/loomcli/landingdeps_integration_test.go`
- `internal/loomcli/parentreview_integration_test.go`
- `internal/loomcli/smoke_driverstrand_test.go`
- `internal/loomcli/smoke_helpers_test.go`
- `internal/loomcli/smoke_starttail_test.go`
- `internal/loomcli/wiring_commitstatus_integration_test.go`
- `internal/loomcli/wiring_publish_integration_test.go`
- `internal/loomcli/wiring_rework_integration_test.go`
- `internal/orchcli/notify_integration_test.go`
- `internal/orchcli/refusal_integration_test.go`
- `internal/orchcli/smoke_live_test.go`
- `internal/pairteardown/teardown_integration_test.go`
- `internal/preflight/preflight_integration_test.go`
- `internal/preflightshed/preflight_integration_test.go`
- `internal/reedcli/naming_integration_test.go`
- `internal/reedcli/smoke_test.go`
- `internal/reedcli/watchdog_integration_test.go`
- `internal/reedcli/watchdogreap_integration_test.go`
- `internal/shedcli/parity_test.go`
- `internal/shedcli/seed_integration_test.go`
- `internal/webstercli/reset_test.go`

**Intent:**
D4, hubforge part: the test hub's side-named accessors and fields take the neutral code/records names, so no test outside fabric's owner set spells a side.

| Old | New |
| --- | --- |
| `Hub.PairWarpWorktree` | `Hub.PairCodeWorktree` |
| `Hub.PairWeftSibling` | `Hub.PairRecordsSibling` |
| `Hub.PrimeWeft` | `Hub.PrimeRecords` |
| `Hub.WarpBare` field | `Hub.CodeBare` |
| `Hub.WeftBare` field | `Hub.RecordsBare` |

Each is renamed with an AST-aware rename over the whole module, every build tag included, never a text replace; the Edit list is every file that spells an old name today.
A grep pass then confirms no old spelling survives in code or comments, `internal/hubforge/doc.go` and `seed.go` included.
Each renamed symbol's own doc comment opens with its new name.

A pure rename: behaviour is unchanged, and the existing hubforge-backed tests are its coverage.

**ImpactSummary:** test-only API: every hubforge-backed test changes mechanically, and no production package imports hubforge.
