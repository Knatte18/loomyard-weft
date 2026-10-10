# Card 6 — drop-ticker-period

**Delete:**
- `internal/reedengine#tickerPeriodFor`

**Edit:**
- `internal/reedengine/watchloop_test.go`

**Intent:**
Decision D5, the cleanup half: card 5 replaced `watchLoop`'s ticker with the re-armed `wake` timer, which removed `tickerPeriodFor`'s only production caller, so the function goes.
`TestWatchState` drops its `TickerPeriodForAnswersPerModeCadence` step and the `tickerPeriodForAnswersPerModeCadence` helper; the per-mode cadence it pinned is now the poll wait's `PollWakeChangedPerOutcome` step and the loop tests card 5 adjusted.

**ImpactSummary:** No production caller remains after card 5, and the one test step that called it goes with it; nothing outside `watchloop.go` and `watchloop_test.go` names the function.
