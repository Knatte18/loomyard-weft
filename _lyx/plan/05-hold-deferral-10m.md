# Card 5 — A held chair's notice is deferred at most 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:** Decision D7: `holdNoticeMaxDeferral` drops from 20 to 10 minutes, halving how long a wedged advisor hides a chair stall while leaving about three times the longest measured advisor batch.
`holdNoticeSettle` and the `HoldDeferral` predicate are unchanged.
The constant's doc comment keeps its reason and gains none of the measured numbers.

`TestWait_HeldTurnEndNotice`'s row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" asserts the notice at the first tick at or past `10 * time.Minute`, a literal rather than the constant, so a change to the cap fails it.
With its 5-minute sleep jump that is the tick at 10 minutes.

**ImpactSummary:** One constant read only by `hold.go`'s deferral check and that test row; `doc.go` and `spec.go` name it without its value, so no doc changes.

**Commit:** 5: Defer a held chair's notice at most 10 minutes
