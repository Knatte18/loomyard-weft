# Card 5 — `holdNoticeMaxDeferral` is 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:** Decision D7.

`holdNoticeMaxDeferral` in `internal/shuttleengine/hold.go` drops from 20 to 10 minutes; `holdNoticeSettle` and the `HoldDeferral` predicate are unchanged.
Its doc comment keeps its rationale.

`TestWait_HeldTurnEndNotice`'s row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" asserts the notice at the first five-minute tick at or past 10 minutes against the literal `10 * time.Minute`, not the constant, so the row fails if the cap moves.

**ImpactSummary:** One constant; its only reader is the deferred-hold settle in `hold.go`, and the seats chair's held-turn notice now arrives at most 10 minutes after the turn end.

**Commit:** 5: Cap a held-turn notice's deferral at 10 minutes
