# Card 5 — fabricengine's side-named exports take the code/records names

**Edit:**
- `internal/battencli/lifecycle_integration_test.go`
- `internal/configcli/configcli_integration_test.go`
- `internal/fabriccli/adoptremote_integration_test.go`
- `internal/fabriccli/cleanhub_integration_test.go`
- `internal/fabriccli/cleanupremove_integration_test.go`
- `internal/fabriccli/clone.go`
- `internal/fabriccli/fabric.go`
- `internal/fabriccli/weft_verbs.go`
- `internal/fabricengine/add.go`
- `internal/fabricengine/add_remoteleftover_integration_test.go`
- `internal/fabricengine/add_rollback_adopt_test.go`
- `internal/fabricengine/add_runrecords_integration_test.go`
- `internal/fabricengine/archive.go`
- `internal/fabricengine/boardweft.go`
- `internal/fabricengine/branchname.go`
- `internal/fabricengine/branchname_test.go`
- `internal/fabricengine/checkout.go`
- `internal/fabricengine/checkout_index_refresh_test.go`
- `internal/fabricengine/checkout_rollback_test.go`
- `internal/fabricengine/cleanup.go`
- `internal/fabricengine/cleanup_primary_integration_test.go`
- `internal/fabricengine/cleanupremotewarp.go`
- `internal/fabricengine/cleanupremotewarp_integration_test.go`
- `internal/fabricengine/clone_adopt_test.go`
- `internal/fabricengine/clone_emptyweft_integration_test.go`
- `internal/fabricengine/clone.go`
- `internal/fabricengine/coalesce_integration_test.go`
- `internal/fabricengine/committedfile.go`
- `internal/fabricengine/committedfile_integration_test.go`
- `internal/fabricengine/commitweftpaths.go`
- `internal/fabricengine/commitweftpaths_test.go`
- `internal/fabricengine/configchanges.go`
- `internal/fabricengine/configchanges_integration_test.go`
- `internal/fabricengine/config_driven_junctions_integration_test.go`
- `internal/fabricengine/destroy.go`
- `internal/fabricengine/destroyremote_integration_test.go`
- `internal/fabricengine/destructivegaps_integration_test.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/dotlyxjunction_integration_test.go`
- `internal/fabricengine/drift.go`
- `internal/fabricengine/export_test.go`
- `internal/fabricengine/fabric.go`
- `internal/fabricengine/fabric_test.go`
- `internal/fabricengine/forkrecords.go`
- `internal/fabricengine/healthreason_integration_test.go`
- `internal/fabricengine/hook_test.go`
- `internal/fabricengine/junction.go`
- `internal/fabricengine/junction_pattern_integration_test.go`
- `internal/fabricengine/junction_repoint_test.go`
- `internal/fabricengine/livestate_verbs_test.go`
- `internal/fabricengine/mergecrucible_integration_test.go`
- `internal/fabricengine/mergeguards.go`
- `internal/fabricengine/mergesiblings_integration_test.go`
- `internal/fabricengine/mergestateactive.go`
- `internal/fabricengine/mergestate.go`
- `internal/fabricengine/open.go`
- `internal/fabricengine/origin.go`
- `internal/fabricengine/origin_integration_test.go`
- `internal/fabricengine/origin_test.go`
- `internal/fabricengine/pairbranch_archive_integration_test.go`
- `internal/fabricengine/prune_dirty_integration_test.go`
- `internal/fabricengine/prune.go`
- `internal/fabricengine/prune_unowned_integration_test.go`
- `internal/fabricengine/pull_integration_test.go`
- `internal/fabricengine/pushanchored.go`
- `internal/fabricengine/pushanchored_integration_test.go`
- `internal/fabricengine/ready.go`
- `internal/fabricengine/ready_test.go`
- `internal/fabricengine/reconcile.go`
- `internal/fabricengine/reconcile_stale_registration_test.go`
- `internal/fabricengine/reconcile_stale_removal_test.go`
- `internal/fabricengine/refscanner.go`
- `internal/fabricengine/refscanner_test.go`
- `internal/fabricengine/remoteleftover.go`
- `internal/fabricengine/remove_archive_integration_test.go`
- `internal/fabricengine/remove_finish_integration_test.go`
- `internal/fabricengine/remove.go`
- `internal/fabricengine/remove_junctions_integration_test.go`
- `internal/fabricengine/remove_records_integration_test.go`
- `internal/fabricengine/remove_refusal_remedy_integration_test.go`
- `internal/fabricengine/removeremote_integration_test.go`
- `internal/fabricengine/remove_siblingdirty_integration_test.go`
- `internal/fabricengine/remove_warpbranch_integration_test.go`
- `internal/fabricengine/resetpairwarp_integration_test.go`
- `internal/fabricengine/snapshot_integration_test.go`
- `internal/fabricengine/status.go`
- `internal/fabricengine/topology.go`
- `internal/fabricengine/unwire.go`
- `internal/fabricengine/unwire_test.go`
- `internal/fabricengine/warpclean.go`
- `internal/fabricengine/warpjunction_test.go`
- `internal/fabricengine/weftbranch.go`
- `internal/fabricengine/weftpaths_test.go`
- `internal/fabricengine/weftwiring.go`
- `internal/fabricengine/worktreelist.go`
- `internal/hubforge/hub.go`
- `internal/hubforge/hub_test.go`
- `internal/hubgeom/parent_test.go`
- `internal/ideengine/spawn_hub_integration_test.go`
- `internal/loomcli/smoke_helpers_test.go`
- `internal/loomcli/smoke_prebootstrap_integration_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/wiring_parentbranch_integration_test.go`
- `internal/preflight/preflight_integration_test.go`
- `internal/shedcli/seed_test.go`
- `internal/webstercli/reset.go`
- `internal/webstercli/sync_integration_test.go`
- `internal/websterengine/audit_test.go`
- `//tools/tokencount/main.go`

**Prosa:**
- `docs/shared-libs/lyxcwd.md`

**Intent:**
D4, fabricengine part: the exported functions and the one `Fabric` method a non-owner file spells take the neutral code/records names.

| Old | New |
| --- | --- |
| `WeftWorktree` | `RecordsWorktree` |
| `WeftWorktreePath` | `RecordsWorktreePath` |
| `WeftBranchName` | `RecordsBranchName` |
| `WeftRepoRoot` | `RecordsRepoRoot` |
| `CommitWeftPaths` | `CommitRecordsPaths` |
| `WarpLyxLink` | `CodeLyxLink` |
| `Fabric.ResetPairWarp` | `Fabric.ResetPairCode` |

Each is renamed with an AST-aware rename over the whole module, every build tag included, never a text replace, so every caller follows, `tools/tokencount/main.go` and `internal/webstercli/reset.go` included; the Edit list is every file that spells an old name today.
The result-struct fields that share the `WeftWorktree` spelling are not part of this card; Card 7 renames them.
A grep pass then confirms no old function name survives in code, comments or docs, `internal/fabricengine/doc.go`, `docs/shared-libs/lyxcwd.md` and non-owner test comments such as `internal/websterengine/audit_test.go`'s included.
Each renamed symbol's own doc comment opens with its new name.
None of the new names exists in the package today.
File names (`commitweftpaths.go`, `resetpairwarp_integration_test.go`) are fabric-internal and stay.

A pure rename: behaviour is unchanged, and the existing tests are its coverage.

**ImpactSummary:** seven exported names change; every caller, in fabric, webster, tools/tokencount and tests, follows mechanically in this commit.
