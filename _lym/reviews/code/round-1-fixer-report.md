# Fixer-report, round 1

## Changes

- **F1 (fixed as proposed):** `internal/reedengine/spawn_test.go` — `TestLaunchStrandLocked_WriteFailureSendsTheFullLine` and `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls` each gain the doc line "It captures the process-global logger, so it does not run in parallel." above the `//testtiming:keep` line.
  Premise checked: the first calls `logcapture.CaptureVerbose`, the second `logcapture.Capture` in its subtests, and neither is parallel.
  Commit `8a3694ac`.
- **F1, wider set (found through the self-check):** `internal/reedengine/panebin_test.go` — the same line added to `TestComposePaneLaunchLine` and `TestComposeSwitchCommand`.
  The branch diff rewrote both (`git diff main...HEAD`), both are serial through `logcapture.CaptureVerbose`, and neither named the state, so they fall under F1's premise.
  Placed in the doc comment, above `//testtiming:keep` where the test carries one.
  Gate: `lym gate test ./internal/reedengine` tier0, all green.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings; it could not find the PATTERN wording for the serial comment, which does not change the disposition.
- fix-affected-set: one observation, LOW, no finding raised: nine other test files hold serial `logcapture` tests with no comment.
  Disposition: fixed in part. I checked the list against the branch diff: `panebin_test.go` is the only file in it whose serial tests the diff rewrote, and I fixed those two tests.
  The others were not rewritten by the diff; see Deferred.

## Disputed

None.

## Deferred

- Serial `logcapture` tests the diff did not rewrite, without a comment naming the state, among them `TestStageLaunchScript_WriteFailureDegrades` in `panebin_test.go` and tests in `attach_test.go`, `launchscript_cleanup_test.go`, `lifecycle_test.go`, `namerepair_test.go`, `probe_test.go`, `reconcile_test.go` and `windowsize_test.go` (the fork's list, not verified).
  Reason: F1 and its location cover the tests this diff rewrote; commenting the rest is a package-wide sweep the reviewer did not ask for.
  If the acceptance criterion covers every serial test of the package, that sweep is the orch's call.
