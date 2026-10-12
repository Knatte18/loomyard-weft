# Card 2 — Hold notice deferral caps at 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:** D7.
`holdNoticeMaxDeferral` drops from 20 to 10 minutes; `holdNoticeSettle` and the `HoldDeferral` predicate are unchanged.
Its comment keeps one sentence per line and states the new reason: advisor turns measured from about 1.5 to 3.5 minutes, so 10 minutes is about three times the longest observed batch.
What it lets through: a chair stall behind an advisor `advisorWork.outstanding` keeps true now reaches the parent after 10 minutes instead of 20, so a legitimately long advisor turn over 10 minutes reaches the parent as a stall notice.
The bound is the cap itself, applied in `settleDeferredHold` whatever the predicate reports.
The existing row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" asserts against the constant, so it would pass for any value; its `check` is rewritten to compare against a literal `10 * time.Minute` (the first tick at or past 10 minutes with `sleepJump` 5 minutes is the tick at exactly 10 minutes), which pins the value the acceptance criterion names.
No other row reads the constant: the settle-interval rows and the predicate-turns-false row stay as they are.
`doc.go` and `spec.go` name the constant and not its value, so they need no edit.

**ImpactSummary:** One constant read in `settleDeferredHold` only; the one table row that asserted against it is re-pinned to a literal, and no other package reads the value.
