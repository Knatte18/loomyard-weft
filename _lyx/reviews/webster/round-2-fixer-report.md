# Round 2 fixer report

All three findings fixed, one commit each, nothing pushed.

## Fixed

### F1 (MEDIUM) — no row for a timed-out half whose stop fails

Commit `10733b4e2`.
Added a row to `TestEngine_Run_RoundFailureRules`: a reviewer that timed out and cannot be stopped returns `ErrHalfNotStopped` with the way forward naming the reviewer's guid, and both halves' removals are asserted.
The review's suggested row (fixer timed out after the handoff) is not deterministic with the fake shuttle: the fixer can end before `join` has taken the reviewer's end, which makes the first `select` stop the reviewer instead of the loop.
The row instead uses a reviewer timeout, where the first `select` stops the fixer successfully and the loop's stop of the reviewer is the failing call.
For that, `fakeRemover` gained an optional `errGuid` that limits its error to one guid, and the table gained a matching `removeErrGuid` column.
No standalone test was added.

### F2 (LOW) — join left a half whose wait errored unstopped

Commit `cea44a61c`.
The loop at the end of `join` now stops a half whose `Wait` returned an error as well as a timed-out one, so a failed stop returns `ErrHalfNotStopped` and an error end no longer leaves a half that may still be running.
The `join` comment and the `doc.go` timeout sentence state the second case.
Added a row to `TestEngine_Run_RoundFailureRules`: a reviewer whose wait fails has both guids removed and the shuttle error returned.

### F3 (NIT) — gofmt on the join log call

Commit `2b2cdef07`.
`gofmt -w internal/burlerengine/handoff.go` added the missing space.

## Checks

`gofmt -l internal/burlerengine/` is empty, `go vet ./internal/burlerengine/` is clean, `go test -race -count=5` passes for `internal/burlerengine` and `internal/shedadapters`, `go test` passes for `cmd/lyx` (the test-budget check), and `lyx loom lint-comments` reports ok.

## Disputed

None.

## Deferred

None.
