# Card 19 — reclassify-fabricengine-reconcile-to-worktree

**Edit:**
- `internal/fabricengine/reconcile_empty_anchor_integration_test.go`
- `internal/fabricengine/reconcile_stale_registration_test.go`
- `internal/fabricengine/reconcile_stale_removal_test.go`
- `internal/fabricengine/remoteonly_integration_test.go`
- `internal/fabricengine/remove_archive_integration_test.go`
- `internal/fabricengine/remove_finish_integration_test.go`
- `internal/fabricengine/remove_guard_integration_test.go`
- `internal/fabricengine/remove_junctions_integration_test.go`
- `internal/fabricengine/remove_records_integration_test.go`
- `internal/fabricengine/remove_refusal_remedy_integration_test.go`
- `internal/fabricengine/removeremote_integration_test.go`
- `internal/fabricengine/remove_remotetask_integration_test.go`
- `internal/fabricengine/remove_reserved_integration_test.go`
- `internal/fabricengine/remove_siblingdirty_integration_test.go`
- `internal/fabricengine/remove_warpbranch_integration_test.go`
- `internal/fabricengine/resetpairwarp_integration_test.go`
- `internal/fabricengine/snapshot_integration_test.go`
- `internal/fabricengine/status_pollution_integration_test.go`
- `internal/fabricengine/stencilcommit_integration_test.go`
- `internal/fabricengine/stencilhistory_integration_test.go`
- `internal/fabricengine/syncweft_integration_test.go`
- `internal/fabricengine/unwire_test.go`
- `internal/fabricengine/warpforward_integration_test.go`
- `internal/fabricengine/warplayout_test.go`
- `internal/fabricengine/weftgit_exclude_test.go`
- `internal/fabricengine/weftgit_pathspec_integration_test.go`
- `internal/fabricengine/weftgit_unborn_warp_test.go`
- `internal/fabricengine/worktreelist_test.go`

**Uses:**
- `plan:internal/hubforge#CopyHub`
- `plan:internal/hubforge#SharedHub`
- `plan:internal/hubforge#Shape`

**Intent:** The last third of fabricengine's hub-taking tests, under the same classification as card 17.
`warplayout_test.go` is the one read-only test: it compares `WarpLayoutForForTest` with `lyxcwd.ResolveWorktree` on the prime, both in-process reads, so it takes `SharedHub` of the `.` no-pair shape.
`weftgit_exclude_test.go` iterates `.`, `backend` and `wts/some-task`: the first two take the copy of their shape and the third stays on `NewHub`, since its anchor is outside the set.
Every other test writes and copies.
Pure refactor of fixtures: the existing assertions cover it.

**ImpactSummary:** Fixture construction changes in twenty-eight test files; no production code and no assertion changes.
