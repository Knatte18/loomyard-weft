# Fixer report — discussion round 3

## Changes

- **F1 (MEDIUM, design): fixed as proposed.**
  The premise holds.
  `finalize` in `internal/shuttleengine/wait.go` sets `cleaned := outcome == OutcomeDone && !run.spec.KeepPane` and removes the strand whatever the gate outcome.
  The discussion spec leaves `KeepPane` unset, and the seats chair gets `KeepPane: !isChair` (`internal/seatengine/prompt.go`).
  `shedadapters.SingleLLMProducer` maps a failed gate to `Stuck` (`internal/shedadapters/singlellm.go`).
  `Store.PrepareRound` in `internal/parentreview/store.go` begins a new round unless the latest one holds a reject or a superseding approve.
  So no resume reaches `gate()`'s expiry branch after the halt.
  In D2's Bound, I deleted the "A resume of the halted run…" sentence.
  In its place, D2 now says the finalize removes the strand, so the next `lyx loom start` is a fresh spawn, the request stays open and unexpired in its round, and the spawn works on the round `PrepareRound` begins.
  The Fix's optional consequence is added as one sentence.
  A verdict meant for the halted round refuses until the next round's request opens, and lands on that request after, because `RecordVerdict` writes through `openUnsettled`, which reads the latest round.
  The following "That is accepted" sentence bounds this: the run deadline's finalize already reaches the same end.
- **F2 (LOW, consistency): fixed as proposed.**
  The premise holds: `closures.gate` in `internal/parentreview/gate.go` calls `s.MarkExpired()` and logs the timeout Warn once `WaitBound` has passed.
  D2's Rationale now says the probe marks nothing, and that only the parent-review closures mark a request expired, at a boundary or in `Final`.

## Disputed

None.

## Deferred

None.
