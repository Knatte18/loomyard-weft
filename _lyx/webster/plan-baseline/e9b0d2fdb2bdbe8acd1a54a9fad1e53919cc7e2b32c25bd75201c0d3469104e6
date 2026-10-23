# Card 12 — settle record round start

**Edit:**
- `internal/shedadapters#settleRecord`
- `internal/shedadapters#Bouncer.recordSettle`
- `internal/shedadapters/doc.go`

**Create:**
- `internal/shedadapters#roundReviewStart`
- `internal/shedadapters#SettleRoundStart`
- `internal/shedadapters#TestSettleRoundStart`
- `internal/shedadapters#TestBouncer_SettleRecordsRoundStart`

**Uses:**
- `internal/shedadapters#roundUsage`
- `internal/shedadapters#roundUsagePath`
- `internal/shedadapters#liveSettleRecord`
- `internal/shedadapters#newBouncerFixture`

**Intent:**
D7: Plan-Review's settle record records the start of the round the settle judged, so the guard and the approved settle can tell an amendment written after that start.

`settleRecord` gains `RoundStartedAt string` (`round_started_at`, omitted when empty), RFC 3339 UTC; its source, shuttle's run `CreatedAt`, is written by `time.Now().UTC().Format(time.RFC3339)` and so is whole-second, and the recorded start is floored to the second; an older record without it reads as one with no known round start (PATTERN-tolerant-records).
`roundReviewStart(runDir, round)` reads `round-<round>-usage.yaml` and returns its review half's `start`; it returns the zero time for round 0, an absent or undecodable record, a record with no review half, and a zero start, logging a read or decode failure at Warn, since a missing start only means the start is unknown.
`recordSettle` fills `RoundStartedAt` from it when non-zero; `approveWithoutReview`'s round-0 settle therefore records none, and `markSettled` keeps the field as it keeps the settle time.

`SettleRoundStart(runDir)` returns the live settle record's round start (the record `liveSettleRecord` returns) and true, or the zero time and false when no record is live or the live record has no or a zero round start; a round start that does not parse reads the same as an absent one, logged at Warn, so the guard admits and the commit seam keeps the record.
Its error return carries only a failed read of the settle record itself.
`internal/shedadapters/doc.go` states the field, its source and that a settle with no known round start reads as such.

Tests:
`TestSettleRoundStart`, in package `shedadapters`, writes records with `writeSettleRecord` and asserts: a live record with a whole-second start reads it back; a live record without the field, an older record, reads false; a record that is not live reads false; no record reads false; an unparsable start reads false.
`TestBouncer_SettleRecordsRoundStart` drives a fixture Bouncer through a converged settle of round 1 with a round-1 usage record whose review half started at a fixed whole-second instant and asserts the settle record holds it; a converged settle with no usage record records none; and a skip-seam approval (round 0) records none.

**ImpactSummary:** one unexported record type gains an optional field and `recordSettle` fills it; every existing reader ignores the field, and `SettleRoundStart` is new with no caller until card 14.
