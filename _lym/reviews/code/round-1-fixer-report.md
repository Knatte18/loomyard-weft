# Fixer report, code review round 1

Gate: `lym gate test ./internal/landingshed ./internal/gateslot ./internal/gatecli` passed (tier0, 1411 passed, 0 failed).
Commits group findings that touch the same file, since F1/F5/F7, F3/F4 and F3/F5 share files.

## Changes

- **F1** fixed as proposed, in `internal/landingshed/finalize.go` and `finalize_gate_test.go`.
  - `NewFinalize` now builds its `Finalize` through the new `assembleFinalize(deps, resolver, gate)`, which wires `onFlakyPass`.
  - `newFinalizeGateFixture` builds through `assembleFinalize` too, and `TestFinalize_FlakyGateFailureLands` no longer installs the sink.
  - Removing the wiring turns that test red.
  - Fixed differently from the proposal in one respect: the fixture calls `assembleFinalize`, not `NewFinalize`.
    `NewFinalize` needs a `Shuttle` and an opened fabric the fixture does not have, and `premerge` and `guard` would change every other fixture test.
- **F2** fixed as proposed, in `verifygate_test.go`.
  - Rows added to `TestVerifyGate_RerunsATestFailureOnce`: runner crash, shell start failure, empty log, rerun that times out, rerun skipped.
  - The empty-log row asserts a single verify command and a failing reason (`wantReasonLog: -1`).
- **F3** fixed as proposed, in `internal/gateslot/runlog.go`, `doc.go`, `internal/landingshed/doc.go` and `verifygate.go`.
  - `TierRunFailures` returns false for a `, failed outside any test` line.
  - Its doc, the `gateslot` package doc, the `landingshed` package doc and the `rerunTestFailure` comment state it.
  - `TestTierRunFailures` lost that line from its first row and gained a row for it.
- **F4** fixed as proposed, in `runlog.go` and `internal/gatecli/failurereport.go`.
  - `ReportCutPrefix` and `ReportOutputLine` are exported and used by the producer.
  - Beyond the review: `ReportOutsideAnyTestSuffix` is exported the same way, since the parser now depends on that string too.
- **F5** fixed differently from the proposal, in `finalize_gate_test.go`, `failedtests_test.go` and `internal/gatecli/counters_test.go`.
  - Proposed: assert the body given to the GitHub merge in `TestFinalize_FlakyGateFailureLands`.
    Not made: that needs the whole pull-request fixture for one string.
    Made instead: `TestFinalize_LandingTextsListFlakyFailuresOnce` calls `landingTexts`, which composes the GitHub merge's body and the landing message, and checks both for the heading and a shared failure listed once.
  - `TestTierRunFailures` gained a header-with-no-runs row and a row for a run with a cause line and no failure.
    Proposed: a cause line ending in a colon.
    Not made: the report format cannot tell such a line from a failure line, so a row would pin an accident.
  - `TestCounters` gained a VSTest block past `failureLines` and a failed block closed directly by a `Passed` line.
- **F6** fixed as proposed, in `pattern/PATTERN-gate-slot.md`: the Finalize bullet states the one rerun, and the `FailedTestsCall` bullet points to it.
- **F7** fixed as proposed, in the `Call` doc of `finalize.go`.

## Self-check

- fix-consistency, LOW: `internal/gateslot/doc.go` package doc understated the false cases of `TierRunFailures`. Fixed as proposed.
- fix-proposal, LOW: same finding as above, merged. Fixed.
- fix-code-correctness, LOW: same doc finding, merged. Fixed.
- fix-code-correctness, LOW: the empty-log row of F2 does not bite if the `first.Log == ""` guard is removed.
  `os.ReadFile("")` fails and `rerunTestFailure` returns `first` unchanged, so the guard only skips a warning.
  Fixed differently: no change.
  Deleting the guard would change production code for a LOW test remark.
  The row pins the observable outcome, one verify command and a failing reason.
- fix-affected-set: no findings.

No fork failed to report.

## Gate repair

The comment gate flagged three comments added this round: a wrapped break in `gateslot/doc.go`, in the `Call` doc of `finalize.go`, and in the doc of `TestFinalize_LandingTextsListFlakyFailuresOnce`.
Each is now rewritten as whole sentences, one per line, with no change of meaning.

## Disputed

None.

## Deferred

None.

## Rulings

None.
