# Card 5 — hold-deferral-cap

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:**
A seats chair's held-turn notice is deferred no longer than 10 minutes (decision D7).
`holdNoticeMaxDeferral` drops from `20 * time.Minute` to `10 * time.Minute`; `holdNoticeSettle` and the `HoldDeferral` predicate are unchanged.
Advisor turns measured in this hub's transcripts ran about 1.5 to 3.5 minutes, so 10 minutes leaves about three times the longest observed batch while halving how long a wedged advisor can hide a chair stall.

`TestWait_HeldTurnEndNotice`'s row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" pins the value: it wants one notice at the first tick at or past `10 * time.Minute`, written as that literal rather than read from the constant, and before `10*time.Minute + 5*time.Minute`.

**ImpactSummary:** One constant read only by `settleDeferredHold`'s deferral check; every `Spec.HoldDeferral` run notifies a stuck hold after 10 minutes instead of 20.

**Commit:** 5: Cap a held-turn notice's deferral at 10 minutes
