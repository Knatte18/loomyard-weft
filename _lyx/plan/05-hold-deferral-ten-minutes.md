# Card 5 — The held-turn notice is deferred at most 10 minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:**
D7 of the decision record.
`holdNoticeMaxDeferral` drops from `20 * time.Minute` to `10 * time.Minute`; `holdNoticeSettle` and the `settleDeferredHold` predicate are unchanged, and the constant's comment keeps saying it is the longest a deferred notice waits.
`internal/shuttleengine/doc.go` and `spec.go` name the constant without a number, so neither changes.
What this lets through: a held turn end whose advisor reading `advisorWork.outstanding` keeps true on a live advisor, one stuck mid-turn or one whose up line the sender never lands, now reaches the parent as a notice after 10 minutes instead of 20.
The bound is the same as before: the notice is sent once per held turn end, a held or dead advisor already stops deferring, and a new event or turn start still drops the pending notice.

The existing row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" reads the constant, so it would pass for any value; its check keeps its range shape but with the literals `10 * time.Minute` and `15 * time.Minute` as bounds instead of the constant (the row's 5-minute `sleepJump` puts the first tick at or past the cap inside that range), so it fails if the constant is anything else.

**ImpactSummary:** the constant is read by `settleDeferredHold` alone and by this table's row; the seats engine's predicate and `Spec.HoldDeferral` wiring are untouched.

**Commit:** 5: shuttle: defer a held-turn notice at most 10 minutes
