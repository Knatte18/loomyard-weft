# Card 2 — A held turn end's notice is deferred at most 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:**
Decision D7: `holdNoticeMaxDeferral` in `internal/shuttleengine/hold.go` drops from `20 * time.Minute` to `10 * time.Minute`.
`holdNoticeSettle` and the deferral predicate are unchanged.
The constant's comment keeps its two reasons and adds no number.

`TestWait_HeldTurnEndNotice`'s row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" asserts its notice against the constant, so it would pass at any value.
That row's check states the cap as `10 * time.Minute` instead of `holdNoticeMaxDeferral`: one notice at or past 10 minutes and before 15, the first 5-minute tick at or past the cap, which pins the acceptance criterion that a chair held while an advisor reads mid-turn notifies at the first tick at or past 10 minutes.

**ImpactSummary:** One unexported constant read only by `settleDeferredHold`; every `Spec.HoldDeferral` run sends a stalled hold's notice up to 10 minutes sooner, and `doc.go`, `spec.go` and `seatengine/doc.go` name the constant, not its value, so no doc changes.
