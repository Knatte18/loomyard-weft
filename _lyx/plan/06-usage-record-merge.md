# Card 6 — usage-record-merge

**Create:**
- `plan:internal/shedadapters#mergeRoundUsage` -> `func mergeRoundUsage(runDir string, round int, now func() time.Time, apply func(u *roundUsage)) error`
- `plan:internal/shedadapters#readCompletedRoundUsage` -> `func readCompletedRoundUsage(runDir string, round int) (roundUsage, error)`

**Edit:**
- `internal/shedadapters#roundUsage`
- `internal/shedadapters#newRoundUsage`
- `internal/shedadapters#BurlerProducer.doneExit`
- `internal/shedadapters#computeLensFacts`
- `internal/shedadapters#lensesRunUnderFan`
- `internal/shedadapters#TestBurlerProducer_Call_WritesRoundUsageRecord`
- `internal/shedadapters#TestWriteRoundFacts_LensSection`
- `internal/shedadapters#TestBurlerProducer_Call_ClusterExclusions`

**Prosa:**
- `internal/shedadapters/doc.go`

**Intent:** Give `round-<N>-usage.yaml` the shape D10 decides and the read rule D11 decides, ahead of the Bouncer writing into it.
`roundUsage` gains `Seed []halfUsage` and `Judge []halfUsage`, yaml keys `seed` and `judge`, both `omitempty`, one entry per pass in the existing half shape; and its `Review` and `Fix` fields become `*halfUsage` with `omitempty`, so a record written by a Bouncer alone carries no review or fix key, and a record's review half is present exactly when the key is.
The file name and every existing field keep their names and shapes, so a record the Burler wrote before this card parses unchanged.
`newRoundUsage` sets both halves as pointers.
`mergeRoundUsage` is the one write path with merge: it reads the record if present, calls `apply` on it, and writes it through `writeRoundUsage`; a missing record starts from the zero value, and a record that does not decode is archived through `archiveStaleOutputs` with `now`, logged as a warning, and replaced by a fresh record holding only this writer's part.
`doneExit` writes through `mergeRoundUsage` with the producer's clock, setting `Fan`, `Lenses`, `Review` and `Fix` and leaving `Seed` and `Judge` as read, and keeps logging a failed write without changing the Stuck hand-off.
`readCompletedRoundUsage` decodes through `readRoundUsage` and returns an error naming the round when the record has no review half; `computeLensFacts` and `lensesRunUnderFan` read through it, so a seed-only or judge-only record reads as missing in the facts file's lens section and makes the next round's lens exclusions ineligible, as a missing record does today.
`TestBurlerProducer_Call_WritesRoundUsageRecord` gains rows: an existing record with seed and judge entries keeps them beside the fresh review and fix halves; a malformed existing record is archived aside and the fresh record holds the round's halves alone.
`TestWriteRoundFacts_LensSection` gains a case where the round's record holds a seed entry and no review half, asserting the lens section reports the record unreadable and that an exclusion whose previous round has such a record is dropped.
The existing fixtures that write a record with no review half would now read as missing, so each gains one: the records `TestWriteRoundFacts_LensSection`'s existing subtests write, which expect `Fan:` and `Lenses run:` lines, and the `usageOf` helper of `TestBurlerProducer_Call_ClusterExclusions`, whose union and held-set rows expect eligible exclusions.

**ImpactSummary:** The usage record's two halves become pointers and two lists are added; the Burler's one write site and the two lens readers change, every other reader is a test, and a Burler-written record keeps its on-disk form; the review-half rule breaks the review-less records of `TestWriteRoundFacts_LensSection` and `TestBurlerProducer_Call_ClusterExclusions`, which this card gives a review half.
