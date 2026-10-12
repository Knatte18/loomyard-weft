# Card 2 — hold-deferral-ten-minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:** `holdNoticeMaxDeferral` drops from 20 to 10 minutes (decision D7); `holdNoticeSettle` and the `settleDeferredHold` predicate are unchanged.
The constant's comment keeps its meaning: 10 minutes is about three times the longest advisor batch measured, and still well inside a review or discussion table's timeout.
The existing row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" compares against the constant, which would pass at any value, so the row's two bounds become the literal `10*time.Minute`.
With `sleepJump` of five minutes the row then fails at 20 minutes and pins the first tick at or past 10 minutes (the acceptance criterion about a chair held while an advisor reads mid-turn).
The cap only shortens how long a predicate reporting outstanding work can defer a notice; a held or dead advisor already stops deferring through `advisorWork.outstanding`, which is not touched.
`internal/shuttleengine/doc.go` and the `Spec.HoldDeferral` comment name the constant and state no number, so neither changes.

**ImpactSummary:** One constant read by `settleDeferredHold` alone; the only test reading it is the one edited row.
