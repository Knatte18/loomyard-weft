# Webster round 2 — fixer report

Both findings were validated against the code and fixed, one commit each.

## Fixed

- **F1** (BLOCKING, `6bf282834`): moved the three remaining hand-rolled log captures onto `internal/testkit/logcapture`.
  `boltpullfirst_integration_test.go`'s diverged-skip subtest now uses `logcapture.Capture(t)`.
  `TestStoreHousekeeping` in `housekeeping_integration_test.go` now uses `logcapture.CaptureVerbose(t)`, and the package-local `lockedBuffer` type and its `bytes` and `sync` imports are deleted.
  `internal/stencilstore/reconcile_test.go`'s refusal-log table uses `logcapture.CaptureVerbose(t)` and drops its `bytes` import.
- **F2** (LOW, `1ed9beff6`): added a `requireCleanBoard` fixture method in `boltpullfirst_integration_test.go` that asserts an empty porcelain status and no `rebase-merge` or `rebase-apply` directory.
  The two conflict rows of `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict` that lacked those checks now call it, and the retry row calls it in place of its inline copy.

## Verification

`lyx gate test ./internal/fabricengine --tags integration` and `lyx gate test ./internal/stencilstore` both pass on the final tree, and `go vet -tags integration` is clean for both packages.

## Disputed

None.

## Deferred

None.
