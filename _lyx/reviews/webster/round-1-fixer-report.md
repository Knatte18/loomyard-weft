# Webster-Review round 1 — fixer report

All three findings fixed, one commit each, none pushed.

## Fixed

- **F1 (LOW), `1d2702a97`.**
  Every new or changed comment the review listed now puts one sentence on each line, breaking further only at an independent-clause boundary:
  the `Call` doc comments in `battenshed/create.go` and `teardown.go`, `PrimeLock`'s doc in `deps.go`, `primelockwait.go`'s header, `RunsRootRel`'s doc in `shedrun/paths.go`, `Add`'s doc and its step-8 and step-10c comments in `fabricengine/add.go`, `forkrecords.go`'s header and function doc, and the new test-file comments in `create_test.go`, `recordleak_integration_test.go` and `add_runrecords_integration_test.go`.
  Untouched legacy lines were left alone, except where a changed sentence shared their line.
- **F2 (LOW), `292a14e7e`.**
  Added `TestAdd_DropsParentRunRecords_SubpathAnchor` to `add_runrecords_integration_test.go`, on a `"backend"`-anchored hub: the pair's weft branch is one commit ahead of the fork point, tracks nothing under the anchored root, and has no root on disk.
  A mutation check (dropping the `AnchorRel` join from `forkrecords.go`'s root pathspec in a scratch copy) makes the new test fail, so it discriminates.
- **F3 (NIT), `1ad5ac7ed`.**
  `TestBattenIntegration_SeedChild_IgnoresPrimesCommittedBattenRecords` now asserts the `Worktree-Create` step ends `Done` before stepping `Seed-Child`.

## Verification

`go build ./...`, `go test ./...` and `go test -tags integration ./internal/shedrun/... ./internal/fabricengine/... ./internal/battenshed/... ./internal/battencli/...` all pass after the last fix.

## Deferred

None.
