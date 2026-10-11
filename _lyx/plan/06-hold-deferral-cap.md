# Card 6 — A held turn end's notice is deferred at most 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:**
Decision D7.
`holdNoticeMaxDeferral` drops from 20 to 10 minutes; `holdNoticeSettle` and the seats `advisorWork.outstanding` predicate are unchanged.
Its doc comment keeps the two-sided rationale, restated for the new value: long enough for an advisor to finish an ordinary batch, about three times the longest advisor turn measured, and short enough that a wedged advisor hides a chair stall for no more than 10 minutes.

`TestWait_HeldTurnEndNotice`: the row `a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral` asserts the notice's elapsed time against a literal `10 * time.Minute` instead of the constant, so the value itself is pinned: one notice at the first tick at or past 10 minutes, before 15 minutes under the row's 5-minute sleep jump.

No doc or comment elsewhere states the 20-minute value; `internal/shuttleengine/doc.go` and `spec.go` name the constant only.

**ImpactSummary:** a `Spec.HoldDeferral` run, the seats chair, sends a held turn end's notice after at most 10 minutes instead of 20; nothing else reads the constant.

**Commit:** 6: Defer a held turn end's notice at most 10 minutes
