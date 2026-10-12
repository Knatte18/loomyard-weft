# Card 16 — batten-sleep-clock

**Delete:**
- `internal/battenshed#sleepGap`
- `internal/battenshed#sleepGapTolerance`
- `internal/battenshed#TestSleepGap`

**Edit:**
- `internal/battenshed#childWait`
- `internal/battenshed#childWait.observeClocks` -> `func (w *childWait) observeClocks(wall time.Time, awake time.Duration)`
- `internal/battenshed#childWait.awakeSince` -> `func (w *childWait) awakeSince(now, at time.Time, paused []pauserecord.Interval) time.Duration`
- `internal/battenshed#innerRunProducer.judgeAgents`
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#TestNotice_QuietOnlyForRunningChildWithLiveDriver`
- `internal/battenshed/doc.go`
- `internal/battenshed#battenshedAllowedImports`

**Intent:**
Move batten's awake-versus-wall sleep detection onto the shared `pauserecord.SleepWatch` (D9) and fix why the 2026-10-12 quiet notice counted almost eleven hours of a suspend.

The cause: `observeClocks` kept one running total of slept time and dropped it, without adding the new gap, whenever the newest activity stamp differed from the one it was counted against.
An agent that wrote between batten's last check before the suspend and the suspend itself (an API error as the network went down is the usual case) moved the stamp, so the first check after the wake read a moved stamp, dropped the total and the night's gap with it, and measured the idle time from that pre-sleep activity across the whole night.
The same drop happened when the newest stamp moved backwards, as when the newest run left the live set.

Fix: `childWait` replaces `clocksSeen`, `prevWallNanos`, `prevAwake`, `asleep` and `asleepStamp` with a `pauserecord.SleepWatch`.
`observeClocks` feeds it the check's wall and awake readings and no longer takes a stamp; `awakeSince(now, at, paused)` is `pauserecord.AwakeSince(at, now, w.sleep.Sleeps(), paused)`, so every activity time keeps exactly the slept time after it, whichever way the stamp moved.
`judgeAgents` calls both with the new signatures, passing nil paused intervals until card 17 passes the coverage's; its returned stamp still keys the notice episode.
`sleepGap`, `sleepGapTolerance` and `TestSleepGap` go: their rule lives in `pauserecord`, and card 2 carries the test rows.
`InnerRunDeps.Awake`'s doc and `doc.go`'s idle-clock sentence describe the interval rule.
`battenshedAllowedImports` gains `github.com/Knatte18/loomyard/internal/pauserecord`.

Test: `TestNotice_QuietOnlyForRunningChildWithLiveDriver` gains the subtest `AnActivityWrittenJustBeforeTheSleepKeepsTheGap`: at the second check's sleep the run's newest activity moves to the clock's now, then the harness suspends three hours, and the quiet window passes after the wake; exactly one quiet notice goes, reporting 45m and not 3h45m.
It fails on today's code, which drops the gap with the moved stamp; the existing sleep subtests, including `AMovedNewestActivityDropsTheGapObservedWithIt`, whose activity lies after the wake, keep passing.

**ImpactSummary:** battenshed-internal: two childWait methods re-signed with their one caller, judgeAgents, on this card; the sleep helpers and their test move to pauserecord.
