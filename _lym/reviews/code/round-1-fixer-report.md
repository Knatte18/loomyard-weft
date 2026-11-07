## Changes

Every finding's premise held against the code. All six were fixed as proposed.
Tier 0 and tier 1 of `internal/driftengine` and tier 0 of `internal/shedrecipe` pass.

- F1: `unchangeableFindings` in `internal/driftengine/check.go` now leads with the report's records and seal fault lines.
  `Findings()` and the Terminal text share a new `faultLines` helper.
  Two rows of `TestDriftGateUnchangeableCells` pin a records fault and a seal fault beside unchangeable cells.
- F2: a `diff: command` test in `verify_test.go` marks `a.txt` under `metadata.files`.
  One row stores a non-matching ruling and asserts no Stuck.
  The other has the judge rule `does not match` and asserts the ruling counts as matching and is not stored.
- F3: a `TestCheck` row seeds an `entries` amendment holding `expectsMetadata` and asserts no cell, the marks in `report.Metadata` and a passing gate.
- F4: `unchangeableWayForward` no longer ends in `then lym loom resume`; the writer exit's appended resume way forward is the only one.
- F5: `backUpBaseline` clears a stale `baseline.meta` beside an empty baseline.
  `nextFreeBackup` (replacing `nextFree`) picks a number where neither `baseline.N` nor `baseline.N.meta` exists, before the first rename.
  Two `capture_test.go` subtests cover both.
  The `RunCapture` doc and `doc.go` describe the new rule.
- F6: new `TestDriftGateUnchangeableCells` rows for a failed second capture, a cell absent from the second capture, a cell of another kind in it, and `baseHead == head`.
  A new `TestNamesCommit` pins the seven-digit minimum, the non-hex separator, upper case, a split run and an empty commit.
  A `{meta_dir}` render test is in `compare_test.go`.
  `Drift.Head` refusal rows are in `entries_drift_test.go` for each entry and in `entries_gate_test.go` for the `drift` gate entry.

A later gate round found fixed-column wraps and over-long comment lines in the comments this round added or touched.
They are now one sentence per line in `capture.go`, `check.go`, `check_test.go`, `doc.go` and `expectation.go`, with no change to behaviour.

Commits: `ece44d82` (F5), `9280735b` (F2), `936b74c8` (F6, other files), `e6a26b9e` (F1, F3, F4, F6 in `check.go` and its test), and a last commit tightening two F6 rows and the F5 docs after the self-check.

## Disputed

- Fork finding, fix-proposal LOW on F1 (grounds: every fix worse): the Terminal findings can name a `--restore` way forward, the `--entries` way forward and the appended resume.
  Evidence: the `NewDriftGate` doc comment and `doc.go` say the failure's findings name a records or seal fault found beside the cells, and the review's F1 asked for exactly that render.
  Hiding the fault's own way forward would drop the stated contract, and rewording does not change that both are named.
- Fork finding, fix-code-correctness LOW (the `!found` clause of `settleUnchangeable` is redundant, so the "cell absent from the second capture" row cannot tell it from the `Kind` comparison; grounds: premise false for a defect of the fix):
  the clause is existing code the fix did not touch, and the row pins the observable behaviour, an absent cell ends the gate Terminal.
  The testing rules assert behaviour at the public surface, not each internal clause.

## Deferred

- Fork finding, fix-affected-set LOW: `captureFaultWayForward` in `check.go` ends `then lym loom resume`, like the clause F4 removed.
  Reason: the review named only the Terminal stop, and `verify_test.go` (`AFailedCurrentCapture`) pins `lym loom resume` in the capture fault's way forward at Drift-Verify's own stop, which does not pass through the writer exit.
  Whether a failed drift gate's capture fault reaches the writer exit's appended text was not traced, so changing it is a decision for the reviewer or the parent.

## Self-check

- fix-code-correctness, MEDIUM, `sameHead` row did not pin the `baseHead != head` guard: fixed with my own change to the fork's proposal (the row's new value is `abcdef12`, an abbreviation of the baseline head).
- fix-code-correctness, LOW, Kind row passed without the `Kind` comparison: fixed (call 1 now gives `x=`, so both captures' `New` is empty).
- fix-code-correctness, LOW, `!found` clause redundant: disputed, see above.
- fix-consistency, LOW, `RunCapture` doc and `doc.go` still described the old backup rule: fixed.
- fix-proposal, MEDIUM, same `sameHead` finding as fix-code-correctness: merged, fixed.
- fix-proposal, LOW, two way forwards at the Terminal stop: disputed, see above.
- fix-affected-set, LOW, `captureFaultWayForward`: deferred, see above.
