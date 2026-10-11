# Card 5 — A held chair's notice is deferred at most 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:**
D7.
`holdNoticeMaxDeferral` in `hold.go` drops from 20 to 10 minutes; `holdNoticeSettle` and the `HoldDeferral` predicate are unchanged.
Its doc comment keeps saying what the cap is for: long enough for a peer to answer an ordinary batch, short enough that a wedged peer hides a stall well inside a review or discussion table's timeout.

`TestWait_HeldTurnEndNotice`'s row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" asserts against the literal `10 * time.Minute` instead of `holdNoticeMaxDeferral`: one notice, at the first tick at or past 10 minutes and before 15 (its `sleepJump` is 5 minutes).
Reading the constant made the row pass at any cap; the literal pins the decided value, so the row fails at today's 20 minutes.

**ImpactSummary:** One constant; its only reader is `settleDeferredHold`, and the docs that name it (`doc.go`, `spec.go`) name it symbolically and stay true.

**Commit:** 5: A held chair's notice is deferred at most 10 minutes
