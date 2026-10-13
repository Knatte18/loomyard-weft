# Card 5 — A held turn end's notice is deferred at most 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:** Decision D7: `holdNoticeMaxDeferral` in `hold.go` drops from `20 * time.Minute` to `10 * time.Minute`, so a seats chair held while an advisor reads mid-turn reaches its parent as a stall notice after at most 10 minutes.
`holdNoticeSettle` and the `Spec.HoldDeferral` predicate are unchanged.
The constant's doc comment keeps its two reasons and adds the measured basis: advisor batches observed at about 1.5 to 3.5 minutes, so 10 minutes leaves about three times the longest.
`doc.go`, `spec.go` and the seats engine name the constant, never its value, so no other text changes.

`TestWait_HeldTurnEndNotice`'s row `a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral` compares the notice's elapsed time against a literal `10 * time.Minute` instead of `holdNoticeMaxDeferral`: one notice at the first five-minute tick at or past 10 minutes and before 15, so the row fails if the cap moves.

**ImpactSummary:** One constant's value; its only reader is `settleDeferredHold`, and the one test row that pins it now pins the value.

**Commit:** 5: A held turn end's notice is deferred at most 10 minutes
