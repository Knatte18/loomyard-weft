# Card 7 — Result fields, envelope keys and trace detail take the code/records names

**Edit:**
- `internal/fabricengine/status.go`
- `internal/fabricengine/prune.go`
- `internal/fabricengine/reconcile.go`
- `internal/fabricengine/remove.go`
- `internal/fabricengine/checkout.go`
- `internal/fabricengine/coalesce.go`
- `internal/fabricengine/forkrecords.go`
- `internal/fabricengine/pushanchored.go`
- `internal/fabricengine/weftbranch.go`
- `internal/fabricengine/weftgit.go`
- `internal/fabricengine/weftwiring.go`
- `internal/fabricengine/add.go`
- `internal/fabricengine/doc.go`
- `internal/fabriccli/fabric.go`
- `internal/fabriccli/weft_verbs.go`
- `internal/fabricengine/mutation_test.go`
- `internal/fabricengine/pushanchored_integration_test.go`
- `internal/fabricengine/destructivegaps_integration_test.go`
- `internal/fabricengine/hubcontainment_integration_test.go`
- `internal/fabricengine/junction_pattern_integration_test.go`
- `internal/fabricengine/reconcile_stale_registration_test.go`
- `internal/fabricengine/reconcile_stale_removal_test.go`
- `internal/fabricengine/remove_remotetask_integration_test.go`
- `internal/fabricengine/remove_warpbranch_integration_test.go`
- `internal/fabriccli/adoptremote_integration_test.go`
- `internal/fabriccli/hubmutation_integration_test.go`
- `internal/fabriccli/removefields_test.go`
- `internal/hubgeom/geometry_integration_test.go`
- `internal/ideengine/spawn_hub_integration_test.go`
- `internal/preflight/preflight_integration_test.go`

**Prosa:**
- `contracts/stencils/shed/shed-template-driver.md`
- `docs/overview.md`

**Intent:**
D4, result and trace part, all hard cuts with no alias.

- The `WarpWorktree` field of `PairStatus`, `PruneEntry` and `ReconcilePairResult` becomes `CodeWorktree`, JSON `code_worktree`, and their `WeftWorktree` field becomes `RecordsWorktree`, JSON `records_worktree`, since a non-owner test spells it too.
  `fabriccli/fabric.go`'s envelope maps for status, prune and reconcile use the same keys.
  `PairStatus`'s branch fields are spelled by no non-owner file and stay.
- `RemoveResult`'s `WarpBranchDeleted`, `WarpBranchKeptReason`, `RemoteWarpBranchDeleted` and `RemoteWarpBranchKeptReason` become `CodeBranchDeleted`, `CodeBranchKeptReason`, `RemoteCodeBranchDeleted` and `RemoteCodeBranchKeptReason`, with JSON and envelope keys `code_branch_deleted`, `code_branch_kept_reason`, `remote_code_branch_deleted` and `remote_code_branch_kept_reason`; `lyx fabric remove`'s long help names the new keys.
- Every `refDetail` and `recordPushIfAdvanced` call site passes the side as `code` or `records`, so a mutation entry's detail reads `side=code` or `side=records`.
  `PushPairAnchored` names its sides the same way, so a failing side's error reads `push code side at <path>` or `push records side at <path>`.
- `contracts/stencils/shed/shed-template-driver.md`'s stranded-branch rule matches `side=code`.
- `docs/overview.md` names the renamed `remove` keys; `internal/fabricengine/doc.go` names the renamed fields, keys and side values.

Each test asserting an old key, field, detail or error text asserts the new one, found by grepping for `warp_worktree`, `weft_worktree`, `warp_branch_`, `side=warp`, `side=weft`, `push warp side` and `push weft side`; `docs/benchmarks/` is a measurement report and is not rewritten.
The existing tests cover the renamed output: `mutation_test.go` pins the detail, `removefields_test.go` the remove keys, and `pushanchored_integration_test.go` the side errors and details.

What the hard cut lets through, and its bound: a run traced by the old binary carries `side=warp`, which the driver's stranded-branch rule no longer matches, so it escalates as for any `side` it does not recognise.

**ImpactSummary:** four result types' JSON keys, the mutation detail's side values and one stencil rule change; consumers are `fabriccli`'s envelopes, the driver stencil and tests, all changed here.
