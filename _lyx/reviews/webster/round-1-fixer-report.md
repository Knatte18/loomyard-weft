# Webster round 1 — fixer report

Every finding was validated against the code and the plan, and all three are fixed.
After the fixes, `go build ./...`, `go vet` under the integration, tmux and llm tags, `go test ./...`, `go test -tags integration ./...` and `lyx loom lint-comments` all pass.

## Fixed

### F1 [BLOCKING] — a misnamed advisor that cannot be stopped escapes the stop rule

Commits `916f09b40` and `a413d55e3`.
The finding holds: in the advisor loop of `Engine.Run`, an error from `startSeat` that wraps `ErrSeatNotStopped` was recorded as a start error on a handle-less `seatRun`, which left a live strand with nothing to stop it.
`Engine.Run` now checks for `ErrSeatNotStopped` from an advisor's start before it records a start failure.
On a match it stops the notice sender and every advisor already started, then returns the joined error with the way forward, the same way the chair-start path does.
Any other advisor start error is still recorded, and the step goes on without that advisor.
The `Run` doc comment states the exception.
The gate's comment lint flagged the first form of that doc comment, whose first line ended in a comma that read as a hard wrap.
Commit `a413d55e3` splits it into two sentences, one per line, and `lyx loom lint-comments` now passes.

Coverage: the new case has the same assertions as the chair-start-failure test (the error wraps `ErrSeatNotStopped` and names the way forward, every started advisor is stopped once, and the chair never starts).
So it went in as a row of that test, which is renamed `TestEngine_Run_StartFailureEndingTheRunStopsTheStartedAdvisors`, rather than in the advisor start-failure test, whose rows all expect the run to succeed.
The row failed before the fix, with the run never ending, and passes after it.
No new top-level test was added, so the test budget is unchanged.

### F2 [LOW] — `errorExit` over a cancelled context is untested

Commit `8c05b728d`.
The finding holds as a coverage gap.
The behavior is kept: card 9 says an error wrapping `ErrSeatNotStopped` is returned without archiving, and replacing it with the context error would drop the way forward for a seat that may still be live.
`TestMultiLLMProducer_Cancellation` gains two rows that cancel during `Run`:
- a plain run error becomes the context error;
- `ErrSeatNotStopped` is returned wrapped, with no context error.

The rows take the place of the reviewer's suggested `TestMultiLLMProducer_ErrorExits` row, because the cancellation test is the one that already drives cancellation at exit.

### F3 [LOW] — doc.go names a reader of `ReservedMarkers` that does not exist

Commit `583dba6dd`.
The finding holds: `multiLLMEntry` never calls `ReservedMarkers` and reaches the list only through `Table.Validate`.
The `internal/seatengine/doc.go` sentence now says the collision check and the value maps read the one declaration, and that a caller building a table reaches it through `Table.Validate`.
The exported accessor stays, because card 4 names it as the package's exposed list.

## Disputed

None.

## Deferred

None.
