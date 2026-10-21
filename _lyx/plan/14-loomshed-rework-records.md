# Card 14 — loomshed-rework-records

**Edit:**
- `internal/loomshed#PRReworkDeps`
- `internal/loomshed#prRework.checkSeams`
- `internal/loomshed#prRework.recreateReviewRunDirs`
- `internal/loomshed#DarnDeps`
- `internal/loomshed#darnWrite.Call`
- `internal/loomshed#reworkFixture.deps`
- `internal/loomshed#TestPRRework_FirstRoundMovesGenerationZero`
- `internal/loomshed#TestPRRework_NilSeamIsNamedError`
- `internal/loomshed#TestDarnWrite_Call`
- `internal/testkit/envkit#ReworkDeps`
- `internal/shedrecipe#reworkTestEnv`
- `internal/loomcli#loomCLI.wire`
- `internal/loomshed/doc.go`

**Create:**
- `plan:internal/loomshed#LatestRejectReview` -> `func LatestRejectReview(reworkDir string) (path string, rejectedAt time.Time, found bool, err error)`
- `plan:internal/loomshed#TestLatestRejectReview` -> `func TestLatestRejectReview(t *testing.T)`

**Intent:** The two PR-Gate reject routes back into a review segment keep what D5 and D6 need.

PR-Rework's archive moves the Plan-Review and Webster-Review run directories whole into the round's prior generation.
`PRReworkDeps` gains the required seam `KeepReviewRecords func(archived, recreated string) error`; `recreateReviewRunDirs` calls it for each subdirectory this round's archive moved (`planMoves` skips an empty one, and a skipped one gets no call), from the latest prior generation's reviews directory (`LatestArchivedReviewsDir`) into the recreated one, so the settle record, its patch and the generation record stay where the next Plan-Bouncer re-entry compares against them; `checkSeams` names it when unwired.
`loomCLI.wire` fills it in the `reworkDeps` it builds with `shedadapters.CarryRecords`; `loomshed` stays off `shedadapters`.
PR-Rework's exempt skip (`PlanReviewSkippable`) is untouched and still runs ahead of the Bouncer's re-entry, settling through `approveWithoutReview`, which now writes the settle record.

`LatestRejectReview` returns the newest committed-or-not `round-<N>/findings.md` under `reworkDir` whose `record.json` carries `rejected_at`, with that time; no round reads as not found, and an unparseable time is an error.
It is loom's `RejectReview` seam for the Plan-Bouncer amendment.

Darn-Write clears the pending rejection before Darn-Bouncer runs, so `DarnDeps` gains the optional seam `RecordRejection func(PendingRejection) error`, called with the pending rejection before `ClearRejection` on the rejected path; a failure is returned as `ClearRejection`'s is, and nil records nothing, so the darn amendment then names no reject file.

Tests: `TestPRRework_FirstRoundMovesGenerationZero` asserts `KeepReviewRecords` is called once per archived subdirectory with the archived and recreated paths; a row whose `KeepReviewRecords` fails returns its error naming the subdirectory; `TestPRRework_NilSeamIsNamedError` adds the new seam's row; `TestLatestRejectReview` is a table over no round, one round, two rounds (the newer wins), a newer round whose record lacks `rejected_at` (skipped, the older one wins) and a bad time; `TestDarnWrite_Call` adds rows for `RecordRejection` called before `ClearRejection` with the pending findings, its failure returned, and a nil seam skipped.
`reworkFixture.deps`, `envkit.ReworkDeps` and shedrecipe's `reworkTestEnv`, whose PR-Rework rows call the producer and so reach `checkSeams`, fill the new required seam.
`loomcli`'s `reworkParityFixture` builds deps too but never calls the producer, only the rework plan gate, so it needs no change.

**ImpactSummary:** One required and one optional seam on loomshed's PR-Rework and Darn-Write deps; loomcli's wire fills the required one here and the optional one in the next card.

**Commit:** 14: loomshed: PR-Rework keeps the review records, the reject's findings are kept for the amendment
