# Card 31 — review-input-reentry

**Create:**
- `plan:internal/shedadapters#Bouncer.keptReviewInput` -> `func (b *Bouncer) keptReviewInput(settledAt string) (string, error)`
- `plan:internal/landingshed#OverlapNotePath` -> `func OverlapNotePath(scratchDir string) string`
- `plan:internal/landingshed#WriteOverlapNote` -> `func WriteOverlapNote(path string, rec MergeRecord, files []string) error`
- `plan:internal/loomcli#fixerReportInput` -> `func fixerReportInput(scratchDir string) func() (string, time.Time, bool, error)`
- `plan:internal/loomcli#overlapNoteInput` -> `func overlapNoteInput(scratchDir string) func() (string, time.Time, bool, error)`
- `plan:internal/loomcli#TestReviewInputSeams` -> `func TestReviewInputSeams(t *testing.T)`

**Delete:**
- `internal/shedadapters#Bouncer.keptRejectReview`
- `internal/loomcli#darnRejectReview`
- `internal/loomcli#darnRejectReviewFile`
- `internal/loomcli#recordDarnRejection`
- `internal/loomcli#TestDarnRejectReview`

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#generationRecord`
- `internal/shedadapters#Bouncer.startNextGeneration`
- `internal/shedadapters#Bouncer.reenterUnchanged`
- `internal/shedadapters#AmendmentNote`
- `internal/shedadapters#withRejectReview`
- `internal/shedadapters#amendmentRejectRow`
- `internal/shedadapters#TestBouncer_Amendment`
- `internal/shedadapters#TestBouncer_Reentry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `plan:internal/landingshed#Finalize.mergeParent`
- `plan:internal/landingshed#TestFinalize_MergeJudgement`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `contracts/stencils/loom/loom-rubric-code-review.md`

**Extends:**
- `internal/shedadapters/doc.go`

**Intent:** Feed a re-entered review perch its new review input, through the Bouncer's re-entry seam the PR-Gate reject used to fill (D5, D10).
`BouncerConfig.RejectReview` becomes `ReviewInput`, with the same shape: the path of an input file and when it was written.
A newer input now also re-opens an unchanged subject: `reenterUnchanged` does not re-settle a live settle record when `ReviewInput` reports an input written after the record's settle time, so the Call starts a new generation, an amendment under `review_amend_max_bounces`, even when the task diff is unchanged; an unchanged diff with no newer input re-settles with no spawn as before, and a changed one runs an amendment scoped to the change.
`startNextGeneration` keeps the newer input's path in the generation record as `review_input` (a record holding the retired `reject_review` reads as none), and `AmendmentNote` names it as the review input the amendment reviews besides the change.
The PR-Gate reject review input goes: `shedrecipe.Env.RejectReview` becomes `ReviewInput`, which `bouncerEntry` passes to every Bouncer whose subject key is set, and the darn reject review readers go.
On a loom run `wire` fills it with `fixerReportInput`, the merge fixer's latest `merge-in` report under the run's scratch directory, so a merge fixer report newer than the Code-Perch's settled verdict runs an amendment generation over the report (D5); the Code-Review rubric's review input section names the report as an input the perch judges the alignment from.
On a darn run `wireDarn` fills it with `overlapNoteInput`, the overlap note Finalize writes: on a darn large-merge reroute `mergeParent` writes, through `WriteOverlapNote`, the parent's changes to the overlapping files since the merged tip it read, or since the task's merge base where no merge record exists, and the re-entered Darn-Bouncer runs an amendment generation over them (D10).
A darn budget-0 recipe reaches the PR-Review stub instead, so no perch reads the note there.

`TestBouncer_Reentry` gains a row: a live settle record over an unchanged subject with a newer review input starts an amendment generation; with an older input it re-settles.
`TestBouncer_Amendment` renames its reject rows to review-input rows, the amendment note names the input, and a row whose generation record holds the retired `reject_review` key renders a note naming no input.
`TestFinalize_MergeJudgement` gains darn large-merge rows that read the overlap note `WriteOverlapNote` left: with a merge record it lists the parent's changes to the overlapping files since the merged tip, without one since the task's merge base, and a loom large merge and a darn merge that is not large write no note.
`TestReviewInputSeams` pins both seams over a scratch directory: absent file, present file with its time.
The darn wiring test asserts an overlap-only reroute's note reaches Darn-Bouncer's amendment input.

`shedadapters`' `doc.go` re-entry section names the review input in place of the reject.

**ImpactSummary:** The Bouncer's re-entry seam is renamed and widened to re-open an unchanged subject on a newer input; its fillers are loom's and darn's wiring, and the reject readers it replaced go.
