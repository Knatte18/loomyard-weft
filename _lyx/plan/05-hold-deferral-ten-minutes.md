# Card 5 — A held turn end's notice is deferred at most 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:**
Decision D7.
`holdNoticeMaxDeferral` in `internal/shuttleengine/hold.go` drops from `20 * time.Minute` to `10 * time.Minute`.
`holdNoticeSettle` and the `Spec.HoldDeferral` predicate are unchanged.
Advisor turns measured in this hub ran from about 1.5 to 3.5 minutes, so 10 minutes leaves about three times the longest observed batch and halves how long a wedged advisor can hide a chair stall.
The constant's comment ("long enough for a peer to answer an ordinary batch and short enough that a wedged peer hides a stall well inside a review or discussion table's timeout") still holds and is kept.
`spec.go`'s and `doc.go`'s sentences name the constant and no value, so neither changes.

Test: `TestWait_HeldTurnEndNotice`'s row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" stands for a chair held while an advisor reads mid-turn.
Its check now compares against the literal `10 * time.Minute` rather than the constant: one notice, at an elapsed time of at least 10 minutes and under 15 minutes (its `sleepJump` is 5 minutes).
So the row pins the 10-minute cap rather than whatever the constant holds, and at the 20-minute value it would fail.
Update the row's error message to name 10 minutes.

**ImpactSummary:** One constant read only by `Run.settleDeferredHold`'s deferral check, which every `Spec.HoldDeferral` run (the seats chairs) uses; a live advisor that stays outstanding now reaches the parent as a stall notice after 10 minutes instead of 20.

**Commit:** 5: Defer a held turn end's notice at most 10 minutes
