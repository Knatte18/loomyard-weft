# Webster round 2 — fixer report

F1 is fixed in one commit, which is not pushed.
F2 is withdrawn without a change, because its premise proved false during the fix.
After the fix, `go build ./...`, `go vet ./...` and `go test ./...` pass, and `gofmt -l` over the diff's Go files prints nothing.

## Fixed

- **F1** (`cf86eab89`): `internal/shuttleengine/startup_test.go:768` reads `..._NeverReady pins ...` again.
  The file no longer differs from the merge base on that line.

## Withdrawn

- **F2**: the review claimed the merge-base `run_commitstatus_test.go` was gofmt-clean, so card 1's reformat was unrelated churn.
  It is not clean: restoring the merge-base rows made `gofmt -l` (go1.26.0) list the file, and `gofmt -d` produces exactly card 1's three-line `funcProducer` literals and re-aligned `name:` keys.
  Card 1's change is gofmt's own output for a file the card had to format, not a defect.
  I committed the restore, found this on the gofmt check, and dropped that unpushed commit with `git reset --keep HEAD~1`, so the range carries no fix-and-revert pair.

With F2 withdrawn, every remaining whitespace-only line pair in the range is a gofmt re-alignment: six in `shedengine/status.go`, three in `shedverbs/steprecord.go` and two in `shedengine/run_commitstatus_test.go`.

## Deferred

None.
