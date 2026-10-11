# Card 5 — A held chair's notice is deferred at most 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine/hold_test.go`

**Intent:**
Decision D7: `holdNoticeMaxDeferral` drops from 20 to 10 minutes; `holdNoticeSettle` and the `Spec.HoldDeferral` predicate are unchanged.
Advisor turns measured in this hub ran about 1.5 to 3.5 minutes, so 10 minutes leaves about three times the longest observed batch while halving how long a wedged advisor can hide a chair stall.
Its doc comment keeps its two clauses and needs no value in prose.

In `hold_test.go`, the row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" compares against `10 * time.Minute` instead of `holdNoticeMaxDeferral`, so the test pins the new cap: one notice at the first tick at or past 10 minutes and before 15.

**ImpactSummary:** Every `Spec.HoldDeferral` run, the seats chairs, sends a held turn end's notice at most 10 minutes after reading it instead of 20.

**Commit:** 5: A held chair's notice is deferred at most 10 minutes
