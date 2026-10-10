# Round 5 fixer-report

Both findings fixed, each in its own commit.

## Fixed

- **F1** (`5d9cbd0b0`) — `reconcileHubWide` now probes the board's upstream after a failed push, through the new `Bolt.HasUpstream`.
  With an upstream the stamp-absent rule holds as before.
  With none, the push failure is logged at Warn and the walk is not reported skipped, so the stamp is written and a board with no reachable remote is not re-walked on every start verb.
  The commit stays local until the board's next push.
  `walk.go`'s doc comment and `doc.go` state the rule.
  New test `TestEnsure_BoardWithNoRemoteCommitsHubWideConfigAndWritesStamp` removes `origin` and asserts the local hub-wide commit, a clean board, the Warn and a stamp with the running key.
  The `internal/hubreconcile` row in `cmd/lyx/testdata/test-budget.yaml` goes from 17 to 18.
- **F2** (`dc14c48e3`) — `CommitSeededStencils`'s doc no longer cites the per-run seeding pass: the empty-list sentence ends at "taking no lock and running no git at all", and the never-pushes line drops the per-run push rationale.

Checks: `lyx loom lint-comments` passes, as do `lyx gate test --tags integration ./internal/hubreconcile` and `lyx gate test ./internal/fabricengine ./cmd/lyx`.

## Disputed

None.

## Deferred

None.
