# Card 12 — settle record round start

**Edit:**
- `internal/shedadapters#settleRecord`
- `internal/shedadapters#Bouncer.recordSettle`
- `internal/shedadapters/doc.go`

**Create:**
- `plan:internal/shedadapters#roundReviewStart` -> `func roundReviewStart(runDir string, round int) time.Time`
- `plan:internal/shedadapters#SettleRoundStart` -> `func SettleRoundStart(runDir string) (time.Time, bool, error)`
- `plan:internal/shedadapters#TestSettleRoundStart` -> `func TestSettleRoundStart(t *testing.T)`
- `plan:internal/shedadapters#TestBouncer_SettleRecordsRoundStart` -> `func TestBouncer_SettleRecordsRoundStart(t *testing.T)`

**Uses:**
- `internal/shedadapters#roundUsage`
- `internal/shedadapters#roundUsagePath`
- `internal/shedadapters#liveSettleRecord`
- `internal/shedadapters#newBouncerFixture`

**Intent:**
D7: Plan-Review's settle record records the start of the round the settle judged, so the guard and the approved settle can tell an amendment written after that start.

`settleRecord` gains `RoundStartedAt string` (`round_started_at`, omitted when empty), RFC 3339 UTC at nanosecond precision; an older record without it reads as one with no known round start (PATTERN-tolerant-records).
`roundReviewStart(runDir, round)` reads `round-<round>-usage.yaml` and returns its review half's `start`; it returns the zero time for round 0, an absent or undecodable record, a record with no review half, and a zero start, logging a read or decode failure at Warn, since a missing start only means the start is unknown.
`recordSettle` fills `RoundStartedAt` from it when non-zero; `approveWithoutReview`'s round-0 settle therefore records none, and `markSettled` keeps the field as it keeps the settle time.

`SettleRoundStart(runDir)` returns the live settle record's round start (the record `liveSettleRecord` returns) and true, or the zero time and false when no record is live or the live record has no or a zero round start; a round start that does not parse is an error naming the run directory.
`internal/shedadapters/doc.go` states the field, its source and that a settle with no known round start reads as such.

Tests:
`TestSettleRoundStart`, in package `shedadapters`, writes records with `writeSettleRecord` and asserts: a live record with a start reads it back at nanosecond precision; a live record without the field, an older record, reads false; a record that is not live reads false; no record reads false; an unparsable start is an error.
`TestBouncer_SettleRecordsRoundStart` drives a fixture Bouncer through a converged settle of round 1 with a round-1 usage record whose review half started at a fixed instant and asserts the settle record holds it; a converged settle with no usage record records none; and a skip-seam approval (round 0) records none.

**ImpactSummary:** one unexported record type gains an optional field and `recordSettle` fills it; every existing reader ignores the field, and `SettleRoundStart` is new with no caller until card 14.
