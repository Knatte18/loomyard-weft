## Changes

- **F1 (LOW), fixed as proposed.**
  `stepLocked` in `internal/shedengine/run.go` re-applies `RowAt` to `st.CurrentProducer` right after the post-gate status re-read.
  `TestStep_EmptyCurrentProducerOnAGatedFirstRowKeepsTheRowWhenItHalts` in `run_routing_test.go` seeds an empty `current_producer`, a gated first row whose gate holds and a producer returning Stuck.
  It asserts `StepResult.Next` and the persisted `CurrentProducer` are the first row's name.
  I did not run that test with the fix removed; two forks traced that it fails then.
  Commit `addf536b`, with the test in `dc93e159`.
- **F2 (NIT), fixed as proposed.**
  `TestRowAt` in `run_routing_test.go` is a table test with a named row, an empty current with producers, and an empty current with no producers.
  Commit `dc93e159`, which also carries the F1 test, because both tests live in one file.

`lym gate test ./internal/shedengine` passed (tier0).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

All four forks returned a report.

## Disputed

None.

## Deferred

None.
