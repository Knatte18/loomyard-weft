# Card 6 — Publish and Finalize verify through verifytree and never ship a dirty tree

**Edit:**
- `internal/landingshed/verifygate.go`
- `internal/landingshed/deps.go`
- `internal/landingshed/publish.go`
- `internal/landingshed/finalize.go`
- `internal/landingshed/doc.go`
- `internal/landingshed/verifygate_test.go`
- `internal/landingshed/verifygate_integration_test.go`
- `internal/landingshed/publish_verify_test.go`
- `internal/landingshed/finalize_verify_test.go`
- `internal/loomcli/landingdeps.go`
- `internal/loomcli/landingdeps_verify_test.go`
- `internal/loomengine/config.go`
- `internal/loomengine/config_test.go`
- `cmd/lyx/pathrules_test.go`

**Uses:**
- `internal/verifytree/verify.go`

**Intent:**
D9 and the landing half of D7.

`landingshed.Deps` drops `VerifyPendingPath` and `VerifyOutputPath` for one told `VerifyDir`;
`loomcli.landingDeps` fills it with `verifytree.Dir(l.AnchorPath())`.
`loomengine.LoomVerifyPendingPath` and `LoomVerifyOutputPath`, their file-name constants, their `config_test.go` assertions and their `pathrules_test.go` rows go.

`verifyGate` becomes a thin adapter over `verifytree`: it holds the told command closure, `verifytree.NewPaths(deps.WorktreeRoot, deps.VerifyDir)`, and two seams (`dirty` over `verifytree.DirtyPaths`, `verify` over `verifytree.Verify`) so unit tests never spawn git or a shell.
`check` loses its `treeChanged` argument and the pending marker: it calls `Verify` with site label `Publish` or `Finalize` after every merge-in, so verify runs exactly when the tree differs from the last verified tree.
That covers resolved conflicts, new parent commits and a crash after a merge, and skips a no-op merge onto a verified tree.
`StatusFailed` is a Stuck reason naming the exit code and the log path, with the existing fix-forward way forward;
a shell that could not start keeps today's could-not-start reason;
an empty command keeps today's skip-with-warning;
a nil command closure keeps today's silent no-op.

Each producer runs the clean-tree check at three points: before merge-in (after Publish's status commit), after merge-in, and after verify.
A dirty tree at any point is Stuck naming the paths, before anything is pushed (Publish) or merged into the parent (Finalize).
A verify that dirties the tree is a non-hermetic test and halts rather than ships.
In `Finalize`, the retry merge-in after the parent moved runs the same three checks.

`doc.go` describes the clean-tree points and the record-keyed verify in place of the pending marker.

Tests:
`publish_verify_test.go` and `finalize_verify_test.go` (fake seams) replace the marker cases with record-keyed ones, and add: a dirty tree before merge-in, after merge-in and after verify each ends Stuck naming the paths, with `PushBranch` never called (Publish) and the parent opener never called (Finalize).
`verifygate_test.go` covers the adapter's status-to-reason mapping, the empty and nil command, and cancellation.
`verifygate_integration_test.go` (real scratch repo, real `verifytree`) pins: after a merge whose verify was interrupted before the record write, the next Publish runs verify;
a no-op merge onto a verified tree skips verify.
`landingdeps_verify_test.go` asserts `VerifyDir` equals `verifytree.Dir(loc.AnchorPath())`.

**ImpactSummary:** loom's Publish and Finalize rows gain three dirty-tree halts and switch from the pending marker to the shared verified-tree record; two loomengine accessors retire.
