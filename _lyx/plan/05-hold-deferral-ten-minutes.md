# Card 5 — hold-deferral-ten-minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:** A held chair's notice is deferred no longer than 10 minutes (D7).
`holdNoticeMaxDeferral` drops from 20 to 10 minutes; `holdNoticeSettle` and the `advisorWork.outstanding` predicate are unchanged, and `settleDeferredHold` reads the constant as before.
What it lets through: a legitimately long advisor turn over 10 minutes now reaches the parent as a stall notice, and the bound is the same one notice per held turn end.
The row of `TestWait_HeldTurnEndNotice` named "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" compares the notice's elapsed time against the constant itself, so it would pass at any value.
Its check is rewritten to compare against the literal `10 * time.Minute`, asserting one notice whose elapsed time is at least ten minutes and under ten minutes plus its five-minute tick, so the first tick at or past 10 minutes is what notifies and a revert to 20 minutes fails it.
The constant's doc comment keeps naming no figure, and `doc.go` names the constant only, so no doc changes.

**ImpactSummary:** One constant read by `settleDeferredHold` and one test row; no other caller or documented figure names the old value.

**Commit:** `5: hold-deferral-ten-minutes`
