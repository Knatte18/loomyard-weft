# Plan-Review round 2 — fixer report

Both findings were validated against `internal/reedengine/watchloop_test.go` and fixed in card 5's Tests paragraph.
`lyx loom validate-plan` passes, with only the two informational `create-new-unit` notes it carried before.

## Fixed

- **F1** (LOW): confirmed against `watchloop_test.go:808-840`: the dormancy test stays in poll mode on unchanged ticks and reads the log after a 20 ms stability window, which a ceiling at or above 20 ms can satisfy by waiting.
  Card 5 now names the value, `PollCeiling: 2 * PollCycle` (10 ms), strictly below the 20 ms window rather than equal to it, since a window equal to the ceiling can still miss a tick by its round-trip time.
  The card records the sweep the finding asked for: the five poll-mode loop tests whose ticks double the wait, each with its timing windows, plus why `TestWatchLoop_UndecidedProbeDoesNotGuess` and the signal-mode tests (`waitForPromotion`) are unaffected.
- **F2** (LOW): confirmed; a count under the ideal fixed-cadence count also admits a fixed cadence slowed by latency.
  `TestWatchLoop_PollModeByDefault` now runs on its own timing copy with a 40 ms ceiling, sleeps a 200 ms window after its three-tick wait, and bounds the window's `list-panes` growth at twelve ticks' worth, against about six backed-off ticks and about forty fixed ones.
  The card names a tick's worth as a named constant, since a re-apply's `list-panes` calls per tick were not pinned from the plan.

## Disputed

None.

## Deferred

None.
