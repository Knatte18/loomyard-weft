# Card 16 — Plan-Review skip and PR-Rework archive see the amendment

**Edit:**
- `internal/loomshed#PRReworkDeps`
- `internal/loomshed#PlanReviewSkippable`
- `internal/loomshed#prRework.archive`
- `internal/loomcli#reworkCommitPathspec`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#ReworkDeps`
- `internal/loomshed#reworkFixture.deps`
- `internal/loomshed#TestPlanReviewSkippable`
- `internal/loomshed#TestPRRework_FirstRoundMovesGenerationZero`
- `internal/loomshed/doc.go`

**Create:**
- `plan:internal/loomcli#TestReworkCommitPathspec` -> `func TestReworkCommitPathspec(t *testing.T)`

**Uses:**
- `plan:internal/loomshed#RemoveAmendment`
- `plan:internal/loomcli#amendmentPathspec`
- `plan:internal/loomengine#LoomAmendmentRecord`

**Intent:**
D7: while the amendment record exists, Plan-Review's skip seam answers false, and PR-Rework's archive of a generation removes the record.

`PRReworkDeps` gains `AmendmentPath string`, the absolute amendment record; empty means the run has none to read.
`PlanReviewSkippable` answers false, before any other read, when the file at `AmendmentPath` exists, so the fixed plan of an exempt rework generation is judged and its settle records a round start; a stat error other than not-exist is returned as its other read errors are, and the Bouncer then reviews for real.
`prRework.archive` removes the record with `RemoveAmendment` after archiving Webster's record, since a new generation's plan replaces the amended one; a removal error is returned like the archive's other steps.
`reworkCommitPathspec` appends `amendmentPathspec` over the fabric's tracked check, so that removal is staged in the round's commit.
`loomCLI.wire` sets `AmendmentPath` to `loomengine.LoomAmendmentRecord(location)` where it builds the rework deps; `envkit.ReworkDeps` and the loomshed tests' `reworkFixture.deps`, which builds the deps `TestPlanReviewSkippable` and the PR-Rework tests run on, set it to a path under their temp dir, absent unless a row writes it.
`internal/loomshed/doc.go` records both rules.

What it lets through: none; it only removes a skip.

Tests:
`TestPlanReviewSkippable` gains rows: an exempt generation that would skip answers false while an amendment record exists, and true again once it is gone; an `AmendmentPath` under a regular file, whose stat fails with a not-a-directory error, is a returned error.
`TestReworkCommitPathspec`, new in package `loomcli` since no test covers `reworkCommitPathspec`, asserts a present amendment record joins the pathspec and an absent untracked one does not.
`TestPRRework_FirstRoundMovesGenerationZero` asserts that an amendment record present before the round is gone after the archive.

**ImpactSummary:** `PRReworkDeps` gains one field set by its two builders, the wiring and `envkit`; `PlanReviewSkippable`'s one caller is the `SkipPlanReview` seam, whose answer changes only while a record exists.
