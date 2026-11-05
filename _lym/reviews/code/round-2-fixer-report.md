# Fixer report — code review round 2

## Changes

- **F1**, fixed as proposed, commit `5a24a396`.
  - `internal/boardengine/doc.go`: the `# Sketch` paragraph now says the Created date stays and the write stamps Revised like any scope change.
  - `_lym/landing/summary.md`: the same correction in the `MakeSketch` bullet.
  - `internal/boardengine/sketch_test.go`, `HeldTaskBecomesASketch`: asserts Created equals its value before `MakeSketch` and Revised is not before it.
- **F2**, fixed as proposed, commit `5a24a396`.
  - `TestRunLock_SketchExemptionIsTheWholeNetChange` gains the row `SketchThatKeepsItsRunStatusIsRefused` with `wantErr` true.
- **F3**, fixed as proposed.
  - `_lym/landing/summary.md`: the "fallback to draft is covered only by reading the code" bullet is deleted.
  - The file is under `_lym`, so it has no commit from this round.

`lym gate test ./internal/boardengine` passed: 678 passed, 0 failed, 2 skipped.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW: the F2 row does not pin the `now.Status == nil` clause. The lens withdrew it in its own report, since the row fails when the clause is dropped. No action.
- fix-proposal, LOW: the F1 assertion `Revised not before Created` cannot detect a missing Revised stamp, because a new entry's Revised equals its Created at whole-second resolution. Deferred, see below.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- fix-proposal's LOW on the F1 assertion: making the test fail when `stampDates` does not restamp Revised needs an injectable clock or a Revised value older than one second, and `boardengine` has no clock seam. Adding one is a production-code change outside these findings. The review asked for exactly the assertion made.
