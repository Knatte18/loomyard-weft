# Card 24 — merge-record-aligned-tip

**Delete:**
- `internal/landingshed#mergeReviewedFile`
- `internal/landingshed#MergeReviewedPath`
- `internal/landingshed#ReadReviewedTip`
- `internal/landingshed#RecordMergeReviewed`
- `internal/landingshed#TestRecordMergeReviewed`

**Edit:**
- `internal/landingshed#MergeRecord`
- `internal/landingshed#mergeBefore`
- `internal/landingshed#MergeIn.readBefore`
- `internal/landingshed#MergeIn.writeRecord`
- `internal/landingshed#Deps`
- `internal/landingshed#sampleMergeRecord`
- `internal/landingshed#recordsEqual`
- `internal/landingshed#newMergeInFixture`
- `internal/landingshed#TestMergeIn_MergeRecord`
- `internal/landingshed#TestMergeRecord_RoundTripAndUndecodable`
- `internal/landingshed_test#newMergeInDepsAt`
- `internal/loomcli#landingDeps`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#LandingDeps`

**Extends:**
- `internal/landingshed/doc.go`

**Intent:** Make the merge record's reviewed tip the aligned tip, the parent tip up to which the merge fixer has read what the parent brought in (D4), and drop the reviewed-tip file the Merge-Review perch wrote.
`MergeRecord.ReviewedTip` becomes `AlignedTip`, serialized as `aligned_tip`; a record an older build wrote under `reviewed_tip` reads with an empty aligned tip, which counts as absent, so the next Merge-In treats it as a first one (`PATTERN-tolerant-records`).
`MergeReviewedPath`, `ReadReviewedTip`, `RecordMergeReviewed` and the file name go, and `Deps.MergeReviewedPath` with them; `landingDeps`, `wire` and the test kit stop filling it.
`readBefore` reads the earlier record's aligned tip and nothing else, keeping an unreadable earlier record as a logged fallback; an absent aligned tip stays empty rather than falling back to the merge base, so card 26 reads it as the fork point.
`writeRecord` carries the earlier aligned tip forward unchanged and computes the overlap from the aligned tip, or from the task's fork point when it is empty, until card 26 makes Merge-In advance it.

The record tests round-trip `aligned_tip`, read an old record's `reviewed_tip` as an empty aligned tip, and pin that the Merge-In record carries the earlier aligned tip forward; a `TestMergeIn_MergeRecord` row over an undecodable earlier record pins the logged fallback to an empty aligned tip.

`doc.go`'s merge-record paragraph names the aligned tip and drops the reviewed-tip file.

**ImpactSummary:** One record field renamed and the reviewed-tip file removed; its writer was the retired Merge-Review approve seam, and its readers are Merge-In and the test kit.
