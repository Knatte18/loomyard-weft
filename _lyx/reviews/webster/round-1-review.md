---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/seatengine/engine.go:147
    summary: an advisor whose strand took a held name and could not be stopped is recorded as a plain start error, so a live strand is left with no handle and ErrSeatNotStopped never surfaces
  - id: F2
    severity: LOW
    class: scope
    location: internal/shedadapters/multillm.go:90
    summary: the errorExit branch that returns ErrSeatNotStopped over a cancelled context has no test, and it departs from the plan's wording that a cancelled context returns the context error
  - id: F3
    severity: LOW
    class: consistency
    location: internal/seatengine/doc.go:21
    summary: doc.go says the recipe entry reads ReservedMarkers, but the entry never calls it, and the exported accessor has no production caller
---

Range: `_lyx/loom/rework/` does not exist and `product.parent` is `main`, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
Build, `go vet` under the integration, tmux and llm tags, `go test ./...` and `go test -tags integration ./...` all pass on the reviewed tree.

### [BLOCKING] A misnamed advisor that cannot be stopped escapes the stop rule

**Location:** `internal/seatengine/engine.go:147` (the advisor loop in `Engine.Run`), fed by `startSeat` at `internal/seatengine/engine.go:187`.
**Issue:** `startSeat` stops a seat whose strand took a held name (`checkName`) and, when that stop fails, returns `errors.Join(err, stopErr)`, which wraps `ErrSeatNotStopped`.
For the chair, `Run` returns that error, so the failure surfaces.
For an advisor, the loop treats every error from `startSeat` as a start failure: it logs a warning, adds the name to `failed`, records `startError: err.Error()` on a `seatRun` with a nil handle, and carries on.
`stopAdvisors` skips a nil handle, so that advisor's strand keeps running with no handle anywhere to stop it, the chair's prompt says it "never started", and the adapter then archives and runs beside a live seat.
Card 6 says "a failed stop there returns `ErrSeatNotStopped` with its way forward", and the Shared Decision `probe-stops-and-adapter-archives` rests on no advisor strand being left without a handle to stop it.
`TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair` covers the held-name row with a successful stop only; `TestEngine_Run_ChairStartFailureStopsTheAdvisors` covers the failed stop for the chair only.
**Fix:** In the advisor loop, when the `startSeat` error wraps `ErrSeatNotStopped`, stop the advisors already started and return it with the way forward, as the chair-start path does; keep the recorded-start-error path for every other error.
Add a row for an advisor with a held name and a failed stop to the advisor start-failure test, asserting the `ErrSeatNotStopped` error and that the earlier advisors are stopped.

### [LOW] `errorExit` over a cancelled context is untested and departs from the plan wording

**Location:** `internal/shedadapters/multillm.go:90`.
**Issue:** Card 9 says a probe, resume or run error returns the context error when the context is cancelled and that an error wrapping `ErrSeatNotStopped` is returned without archiving.
`errorExit` returns the wrapped `ErrSeatNotStopped` even when the context is cancelled, which preserves the way forward but is a behavior the plan does not state.
`TestMultiLLMProducer_ErrorExits` runs every row on a live context and `TestMultiLLMProducer_Cancellation` has no `ErrSeatNotStopped` row, so no test reaches the branch.
**Fix:** Add a row to `TestMultiLLMProducer_ErrorExits` that cancels the context and returns `ErrSeatNotStopped`, asserting the error wraps it.
If the plan wording is meant literally, drop the exception instead.

### [LOW] doc.go and card 11 name a reader of `ReservedMarkers` that does not exist

**Location:** `internal/seatengine/doc.go:21`, `internal/shedrecipe/entries_multillm.go`.
**Issue:** `doc.go` states that "the table's collision check, the seats' value maps and the recipe entry that builds a table all read" `ReservedMarkers`.
The collision check and the value map read the package variable directly, and `multiLLMEntry` never calls `ReservedMarkers`; reserved and blank values reach it only through `Table.Validate`.
The exported accessor is called by tests alone.
Only one declaration exists, so no second list was written and the Shared Decision `reserved-markers-declared-once` holds in effect.
**Fix:** Reword the `doc.go` sentence to say the entry reaches the list through `Table.Validate`, or have the entry read `ReservedMarkers` where it needs the list.

### Checks that raised no finding

- **Probe before archive (cards 7, 9):** `MultiLLMProducer.Call` probes before `archiveStaleOutputs`.
  `Engine.Probe` stops every live advisor through `stopSeat` when the chair is not live, and `seatengine` contains no archive call.
  The adapter test pins a live chair leaving the outputs untouched.
- **Chair-decides rule and the stop paths (card 6):** `Engine.join` stops advisors after the chair's wait on a normal end and on a wait error, and `Engine.Run` stops the started advisors when the chair fails to start.
  The notice sender is stopped before the advisors on both paths.
  F1 is the only exit that leaves a strand without a handle.
- **Shuttle send lock and quiet hold (card 1):** `sendMu` is taken with `defer` in `Run.Send` and `Run.sendWithin`, so it is released on every path.
  `notifyHeld` skips only the notice for `QuietHold`.
- **Handle-shaped seams (cards 4, 8, 9, 10):** `SeatRunner` and `Shuttle` have the burler shape.
  `loomCLI.wire` fills `Seats` and `Models` in the same commit as the `Env` fields, and battencli lists both as intentionally nil.
- **Stencil include form and seat blocks (cards 2, 3):** `FillWith` refuses an include with its own pipeline, a missing include and a nested include, and the marker guarantee covers include bodies.
  The two seat blocks are registered and their marker sets are pinned in `TestStencils_TopLevelMarkers`.
- **Per-card mechanical checks:** no card carries a `Rename` or `Delete` group.
  Every `Edit` and `Create` symbol the cards name exists in the diff, and the budget rows rose by the tests added (stencil +1, shuttleengine +1, shedfake +1, shedadapters +4, shedrecipe +3), with `seatengine` at 15.
- **Test economy:** I found no test that duplicates another's coverage.
  The held-name `Run` call at the end of `TestEngine_Probe_ChairNotLiveStopsTheLiveAdvisorsAndLeavesAFreshStart` overlaps a row of the chair-start-failure test, but card 7 asks for it.
- **Docs in the same commit (card 14):** `docs/overview.md` names `seatengine` and the five adapters, the execution stack carries the `seats` line, and `shed-recipe-spec.md` and the `shedrecipe` and `shedadapters` doc files describe the new row.
  No PATTERN entry is required: the new rules (reserved markers, one-level includes, quiet hold) are local to their packages.
