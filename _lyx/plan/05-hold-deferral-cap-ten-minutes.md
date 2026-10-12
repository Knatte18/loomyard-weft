# Card 5 — hold-deferral-cap-ten-minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:** Decision D7 of the decision record: `holdNoticeMaxDeferral` drops from 20 to 10 minutes.
Its doc comment keeps its two bounds and says why ten: advisor turns measured in this hub ran from about 1.5 to 3.5 minutes, so the cap leaves about three times the longest observed batch while halving how long a wedged advisor can hide a chair stall.
`holdNoticeSettle`, `settleDeferredHold` and the predicate are unchanged.

`TestWait_HeldTurnEndNotice`'s row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" keeps its assertion against the constant and adds that the one notice lands at exactly 10 minutes of run-clock time: its 5-minute sleep jump makes the first tick at or past the cap the cap itself, so the row fails at today's 20 minutes and passes after the change.

**ImpactSummary:** One constant in hold.go, read by settleDeferredHold alone; a chair held while an advisor reads mid-turn notifies at the first tick at or past 10 minutes instead of 20.
