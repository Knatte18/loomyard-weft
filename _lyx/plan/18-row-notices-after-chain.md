# Card 18 — row-notices-after-chain

**Edit:**
- `internal/battenshed#landProducer.stuck`
- `internal/battenshed#worktreeTeardownProducer.Call`
- `internal/battenshed#HandBackNotice`
- `internal/battenshed#TestHandBackNotice`
- `internal/battenshed#TestLand_FailingStepsAreStuckWithOneNotice`
- `internal/battencli#battenCLI.driverDepsFor`
- `internal/battencli#battenCLI.wire`
- `internal/battenshed#TeardownDeps`
- `internal/battenshed#landProducer.pausedReason`
- `internal/battenshed/doc.go`

**Delete:**
- `internal/battenshed#rowNotifier.failed`
- `internal/battenshed#TestWorktreeTeardown_FailureNoticeOncePerEpisode`

**Create:**
- `internal/battenshed#RowStopNotice`
- `internal/battenshed#TestRowStopNotice`
- `internal/battenshed#PauseHonouredPrefix`
- `internal/battencli#battenHandBackNotice`
- `internal/battencli#TestBattenHandBackNotice`

**Intent:**
Hold the Land and Worktree-Teardown failure notices until the stop chain gives the stop back (D8): the orch is told only what the rule table, the tackler and the arbiter did not settle.
`landProducer.stuck` and `worktreeTeardownProducer.Call` no longer post a failure notice: they record the stuck reason and return Stuck as today, and `rowNotifier.failed` goes.
`RowStopNotice` is the notice the driver's hand-back sends for a stop at a row after Run-Shed: `<slug>: <row> failed: <reason cut to one line>; report <reportPath>; way forward: fix the cause, then run "lyx batten run <slug>"`, sent through the row's notifier with its retry spacing and cap, once per stop by the row's `handback <historyLen>` marker line that `HandBackNotice` already keeps, so a restarted driver sends no second.
`battenHandBackNotice` is the router `driverDepsFor`'s hand-back `Notify` calls: it sends nothing for a paused stop, as today, and nothing for a blocked Land stop whose reason starts with `PauseHonouredPrefix`, the pause Land honoured while it waited, which today sends no notice and whose own reason names `lyx batten goto <slug> --to Land` first; it sends `RowStopNotice` with the stop's reason and the report path for any other stop whose current producer is Land or Worktree-Teardown, and `HandBackNotice` otherwise.
`landProducer.pausedReason` builds its text from `PauseHonouredPrefix`, so the router and the reason share one literal.
`HandBackNotice` drops its check for the row's own `failure <n>` marker, which no row writes any more.
With Worktree-Teardown's own notice gone, `TeardownDeps` loses `Notices` and `wire` stops filling it; Land keeps its `RowNoticeDeps` for the records-stop line.
`doc.go`'s row-notice paragraph says the failure notice waits for the hand-back and carries the stop report.
Tests: `TestRowStopNotice` (tier0, parallel) rows: a first hand-back of a Land stop posts the line with the report path, a second call for the same history length posts nothing, a later stop posts again, and a nil Notify posts nothing; `TestBattenHandBackNotice` (tier0, parallel, over a fake `RowNoticeDeps.Notify` recording lines) rows: a failed Land stop posts the row-stop line, a pause-honoured Land stop and a paused stop post nothing, and a stop at Run-Shed posts the `HandBackNotice` line; `TestHandBackNotice` loses its failure-marker row; `TestLand_FailingStepsAreStuckWithOneNotice` asserts no notice from the row.

**ImpactSummary:** Two rows stop notifying, TeardownDeps loses its Notices seam with its one fill in wire, and one driver seam routes their hand-backs to a new notice; the hand-back marker keeps one notice per stop.
