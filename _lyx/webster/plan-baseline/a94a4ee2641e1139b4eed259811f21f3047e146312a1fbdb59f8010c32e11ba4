# Card 18 — skipped-history-readers

**Edit:**
- `internal/loomengine#checkCoherence`
- `internal/loomcli#checklistVisitsOf`
- `internal/reportengine#countHandSteps`
- `internal/battenshed#classifyChanges`
- `internal/loomengine#TestCheckCoherence`
- `internal/loomcli#TestRenderChecklist`
- `internal/reportengine#TestCompute_RepairsAndRounds`
- `internal/battenshed#TestClassifyRecords`
- `internal/reportengine#mergeRows`
- `internal/reportengine#TestComputeTiming`

**Uses:**
- `internal/shedengine#OutcomeSkipped`

**Intent:**
Every history reader treats a `skipped` entry as a row passed through, never as a stop or a call (D8); the caller label stays on the verdict that led to the skip.

`checkCoherence` accepts `skipped` among the history outcomes, so a seeded or resumed run whose history holds a skip is not refused as incoherent; `TestCheckCoherence` gains that row.
The status checklist shows a skipped row as passed through, neither a visit nor a stop, and `TestRenderChecklist` gains a row with a skipped Describe.
The report's hand-step count does not count a skip as a step a hand took, and `TestCompute_RepairsAndRounds` gains a history with a skip.
Batten's notice classification reads a child's skip as no change worth a notice, and `TestClassifyRecords` gains that row.
The timing report's `mergeRows` leaves a `skipped` entry out of the timed rows, so a skip never reads as a call that took the time up to the next entry, and `TestComputeTiming` gains a history with a skipped Describe whose time counts under the row before it.
The timing stops and waits (`runStops`, `unclaimedHaltEntry`, `battenRowHalts`) read only Stuck and Awaiting entries, which a skip never is, and `latestDarnOutcome`, the driver's moved-run check and the relaunch admission read only Darn-Write, goto and the stop key, which a skip never names, so they need no change.

**ImpactSummary:** Five history readers learn the `skipped` outcome; none changes for a history without one.
