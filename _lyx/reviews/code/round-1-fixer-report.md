# Round 1 fixer-report

## Changes

- F2: `providerlessShuttleConfig` (`internal/loomcli/smoke_helpers_test.go`) also sets `start_retries: 0` and its header says the fixtures take one launch; commit `9b845126`.
- F3: the `keyTimesReed` doc line moved from above `TestStartup_DiedStartLaunchesAgain` to above `type keyTimesReed`; commit `16b6ca4c`.
- F4: `TestStartup_DiedStartLaunchesAgain` gained a timeout row (run deadline inside a long startup window, one launch) and a row where every launch dies; it asserts the error names exactly one run directory, the count of `startRetryDelay` sleeps and, for the unremovable strand, the 1 s then 2 s removal waits.
  `TestStartup_PaneNotLiveMidStartup` asserts the removal-retry rows' full run-clock sleeps.
  Commit `16b6ca4c`, refined in the self-check commit below.
  The timeout row sets `StartupTimeoutS: 3600` because a startup-window expiry is classified `died` (`wait.go:725`) and only the run-deadline path yields `timeout`.
- F5: a `runStdin` subtest of `TestTmuxExecRetriesEAGAIN` asserts both calls see the same non-nil stdin; commit `c0b5d3b0`.
- Self-check follow-ups, one commit: the removal-wait assertion compares the whole sleep slice, `providerlessShuttleConfig` panics when the `start_retries: 1` line drifts, and the F5 subtest notes it pins the hook seam only.

F1 is `decision`-class and was handled by the parent (recorded decision: the notice is withheld only for background work); it is not in this report.

`lyx gate test` over `./internal/shuttleengine`, `./internal/reedengine`, `./internal/loomcli` is green.

## Self-check

- fix-code-correctness, LOW, the F5 subtest passes with a non-hook defect in place: fixed differently — the real exec path cannot be made to fail with EAGAIN in a test, so the subtest carries a comment that it pins the hook seam only.
- fix-code-correctness, LOW, the `start_retries` rewrite is silent on drift: fixed — it panics on drift like `claudeConfigWithBinary`.
- fix-consistency, LOW, the same drift finding: merged with the one above, fixed.
- fix-proposal, MEDIUM, F5 row cannot catch the reader-rebuild defect: fixed differently, as above; no exec-path test is reachable.
- fix-proposal, LOW, single-wait removal row passes without the removal sleep: fixed — rows compare the whole sleep slice.
- fix-proposal, LOW, F2 drift: merged with the drift finding, fixed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
