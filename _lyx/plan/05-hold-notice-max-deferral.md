# Card 5 — hold-notice-max-deferral

**Edit:**
- `internal/shuttleengine#holdNoticeMaxDeferral`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`

**Intent:** D7: `holdNoticeMaxDeferral` drops from 20 to 10 minutes.
`holdNoticeSettle` and the predicate are unchanged.
The constant's doc comment keeps its two bounds and gains the measure behind the value: advisor turns measured in this hub ran from about 1.5 to 3.5 minutes, so 10 minutes leaves about three times the longest observed batch while halving how long a wedged advisor can hide a chair stall.
It also states what the cap governs: a reading the predicate keeps true on a live advisor, one stuck mid-turn or one whose up line never lands, since a held or dead advisor already stops deferring.

`TestWait_HeldTurnEndNotice`'s row "a deferred run whose predicate always reports outstanding work notifies once, only past the maximum deferral" pins the value: it asserts the notice came at the first tick at or past 10 minutes, written as the literal `10 * time.Minute` beside the constant, so a later change to the constant fails here rather than passing by symbol.
Its 5-minute sleep jump and 2-hour timeout stay.

**ImpactSummary:** One constant in `hold.go`; `settleDeferredHold` is its only reader, `doc.go` and `spec.go` name it without a number, and the one test row that reaches the cap is updated.
