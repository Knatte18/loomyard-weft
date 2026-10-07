# burler-cost: split a Burler round into a reviewer and a fixer

All three batches recorded `done`, in order.

## Batch 01 — shuttle-probe (head `63a6e8a4e`)

Cards 1 to 4 landed:
- `Runner.ProbeGated` returns AttachGated's probe handle unwaited.
- `burlermarker.Path` is the one review-path-to-marker derivation.
- `review` and `fix` are scalar-or-list keys resolved per round, and the recipe `model`/`effort` keys are removed.
- `lyx burler await-review <marker>` is a read-only, capped poll.

Nothing fell outside the batch's declared file-ops.
The fork also wrapped model-spec load errors with the offending key, and added sandbox scenario S4 for `await-review`.

## Batch 05 — round-stencils (head `d2f5fddfe`)

Cards 5 to 7 landed:
- The reviewer and fixer orchestrator stencils, with the fixer's `Disputed` section.
- `StrandRemover` and `NewReedStrandRemover`.
- `Engine.Run` starts both halves and joins them through the marker, with callers updated.

The one reported deviation is `internal/shedrecipe/fixture_test.go`, where `RunRoot` moved inside `AnchorPath` so the marker derives as it does in production.
The fork also set the marker-released flag before the marker write, so a fast fixer cannot cause a false skipped handoff.
`join` returns at once with `ErrHalfNotStopped` on a failed stop.
`TestRunCmd_PassesWatchTrueToReedUp` became `TestRunCmd` with subtests, and the `internal/burlerengine` test budget row was raised.

## Batch 08 — two-half-resume (head `c6c4d88e5`)

Card 8 landed `ProbeRound` and `Resume`, plus the producer's two-strand probe and retry.
The one reported deviation is `internal/burlercli/wiring_test.go`, where the `diedShuttle` double implements the new `Shuttle.ProbeGated`.
The fork unified `Call`'s done-outcome mapping into `doneExit`, and reports that the rewritten tmux smoke test `smoke_attachprobe_test.go` was vetted but not run.
