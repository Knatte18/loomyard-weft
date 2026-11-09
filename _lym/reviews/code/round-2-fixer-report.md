# Round 2 fixer-report

## Changes

All four findings were fixed as proposed.
F2, F3 and F4 touch the same test files, so they share one commit.

- **F1** (`internal/gatecli/redundancy.go`, `runTest`): right after `RunAlone` returns, a cancelled `s.ctx` returns `reasonCancelled`.
  A test killed in flight is now listed as cancelled, `retries()` is true, and the next call reruns the step.
- **F2** (`redundancy_integration_test.go`): new subtest `ACallCancelledMidRunHoldsASlotUntilThenAndListsTheTestInFlightAndTheRestAsCancelled`.
  It cancels the step's context once `TestDropsHangs` has started, then asserts `TestDropsHangs` and `TestDropsLate` are not judged for `reasonCancelled` and that `retries()` is true.
  The `testmap` stand-in gained `testMapHangingEnv`: a hanging test appends its name to that file, which is how the test knows the run is in flight.
- **F3** (`redundancy_integration_test.go`): the `stale` map now also holds `TestStaleNew`, a judged test, with its current function hash.
  Without the `judged[mapped.Name]` clause it would cover itself and turn from unique to contained, so the first subtest fails.
- **F4** (`redundancy_integration_test.go`): the same new subtest asserts a holder record with `Site` `redundancySite` for the worktree exists while the test runs and is gone after the step ends.
- **Self-check finding** (fix-consistency, LOW): the `runTest` doc now names the cancelled case.
- **Gate findings** (comment line breaks): the `runTest` doc is rewritten as one sentence per line, and the over-long `TestStaleNew` comment in `redundancy_integration_test.go` is split into two sentences.

Commits: `2448e466` (F1), `65618b30` (F2, F3, F4), one for the doc line, and one for the gate's comment repair.
Gate: `lym gate test ./internal/gatecli` (tier 0) and `lym gate test --tier 1 ./internal/gatecli` both exit 0, with the new subtest passing.
The doc-line commit changed a comment only and was checked with `go vet ./internal/gatecli`, not by a gate rerun.

## Disputed

None.

## Deferred

- **fix-consistency finding 2** (LOW): `testMapRun.runTest` in `internal/gatecli/testmappackage.go:182` has the same shape, a test killed by a cancel reads as a failed test.
  The fork itself called it no defect in this change set.
  The review does not cover it, and what a cancelled `lym gate testmap` should write into a map is a behaviour decision for the testmap command, not a fix to the redundancy step.
  Parent to decide whether to open it.

## Rulings

None reached me.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, `runTest` doc omits the cancelled case: fixed as proposed.
- fix-consistency: LOW, `testMapRun.runTest` twin has no cancel check: deferred, see above.
- fix-proposal: no findings.
- fix-affected-set: no findings.

Every fork returned a report; none is "no report".
