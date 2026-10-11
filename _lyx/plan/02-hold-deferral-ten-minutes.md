# Card 2 — hold-deferral-ten-minutes

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:** `holdNoticeMaxDeferral` in `internal/shuttleengine/hold.go` drops from `20 * time.Minute` to `10 * time.Minute` (D7); `holdNoticeSettle` and the `HoldDeferral` predicate are unchanged.
Its comment is rewritten, one sentence per line: the cap leaves about three times the longest advisor batch measured in this hub's transcripts, and halves how long a wedged advisor can hide a chair stall.
It governs only a reading the predicate keeps true on a live advisor, one stuck mid-turn or one whose up line never lands; a held or ended advisor already stops deferring.

`TestWait_HeldTurnEndNotice`'s row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" today bounds its notice by the constant itself, so it passes at any value.
The row's check pins the shipped value: the one notice falls at or past `10 * time.Minute` and before `10*time.Minute + 5*time.Minute` (its `sleepJump`), so the acceptance criterion "a chair held while an advisor reads mid-turn notifies at the first tick at or past 10 minutes" is asserted, not derived.
The row's `timeout` of two hours stays.

`internal/shuttleengine/doc.go`, `spec.go` and `internal/seatengine/doc.go` name the deferral only by its role, never by its length, so none changes.

**ImpactSummary:** One unexported constant read in production only by `settleDeferredHold` in `hold.go`; every `HoldDeferral` run (the seats chair) notices a stall after at most 10 minutes instead of 20, and one test row's bound changes.
