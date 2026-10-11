# Card 5 — hold-deferral-cap

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:**
A seats chair's held-turn notice is deferred no longer than 10 minutes (decision D7).
`holdNoticeMaxDeferral` drops from 20 to 10 minutes; `holdNoticeSettle` and the deferral predicate are unchanged.
Advisor turns measured in this hub ran about 1.5 to 3.5 minutes, so 10 minutes leaves about three times the longest observed batch while halving how long a wedged advisor can hide a chair stall.
What this lets through sooner is a stall notice for a legitimately long advisor turn over 10 minutes; nothing else changes.

`TestWait_HeldTurnEndNotice`'s row for a deferred run whose predicate always reports outstanding work asserts the notice's elapsed time against the literal `10 * time.Minute` instead of the constant, so the row pins the cap's value: one notice at the first tick at or past 10 minutes and before 15.

**ImpactSummary:** one constant; every deferred hold, which only seats chairs use, notices at most 10 minutes after its turn end.

**Commit:** `5: A chair's held-turn notice defers at most 10 minutes`
