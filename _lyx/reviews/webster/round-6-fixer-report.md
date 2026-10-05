# Round 6 fixer report

## Fixed

- F1: `escalate` in `internal/shedadapters/bouncer.go` now appends a way forward to a failed `writeEscalation` error.
  `refusal-spec.md` gained a shed-section row, "escalation record write failed", and `TestBouncer_Escalation_FailedRecordWriteErrorsNamingTheWayForward` makes the notice write fail and asserts the path and `way forward:`.
- F2: the "escalation record unreadable" and "circling decision unreadable" rows now cover read failures as well as malformed records.
  The escalation Reason now says "fix or delete the file the error names" instead of always naming the escalation file, since the error may name the parent notice.

F1 and F2 touch the same lines of `bouncer.go` and the same spec rows, so they landed in one commit, `shedadapters: fix F1, F2`, rather than two.
`gofmt -l` lists nothing, and `go test` passes for `shedadapters` and `contracts/...`.

## Deferred

Nothing deferred.
