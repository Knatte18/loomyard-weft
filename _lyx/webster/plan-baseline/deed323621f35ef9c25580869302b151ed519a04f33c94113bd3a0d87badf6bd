# Card 16 — recipe-merge-seams

**Create:**
- `plan:internal/shedrecipe#colorSegmentKey` -> `func colorSegmentKey(entry string, cfg Config) (segmentcolor.Segment, error)`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#burlerFixEntry`
- `internal/shedrecipe#TestBouncerEntry_SkipSeam`
- `internal/shedrecipe#TestBouncerEntry_ApproveSeam`
- `internal/shedrecipe#TestBouncerEntry_ConstructionFailures`
- `internal/shedrecipe#TestBurlerHalfEntries_StrictUnknownKeys`
- `internal/shedrecipe#TestBurlerReviewEntry_RubricStencil`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli#loomCLI.wire`
- `internal/battencli#intentionallyNil`

**Uses:**
- `internal/shedrecipe#parseSegment`
- `internal/landingshed#ReadMergeRecord`
- `internal/landingshed#MergeRecord.SkipsReview`
- `internal/landingshed#RecordMergeReviewed`
- `internal/landingshed#MergeReviewNote`

**Intent:**
Teach the Bouncer and Burler entries the keys the Merge-Review rows will carry, and fill the seams they resolve, before any recipe row uses them.

`Env` gains:
- `SkipMergeReview func() (bool, error)`, the merge skip seam (D6);
- `ApproveMergeReview func() error`, which records the last finished review (D6);
- `MergeReview func() string`, the rubric's `merge_review` note;
- `MergeReviewOff bool`, `loom.yaml`'s `skip_merge_review`, which card 19's `New` reads.

`bouncerEntry` changes:
- `skip_seam` accepts `merge-overlap`, resolved to `SkipMergeReview` under `requireSeam`, beside `rework-exempt`.
- `approve_seam` accepts `merge`, resolved to `ApproveMergeReview` under `requireSeam`, beside `plan`.
- A boolean `blocking_escalates` key fills `BouncerConfig.BlockingEscalates`, default false.
- A `color_segment` key fills `BouncerConfig.ColorSegment` through `colorSegmentKey`, absent meaning the review colour.
- `Env.MergeReview` reaches `BouncerConfig.MergeReview`.
Each refusal names the accepted values, as today.

`burlerReviewEntry` accepts `color_segment` beside `run_subdir`, sets `Profile.Segment`, and passes `Env.MergeReview` into the `RubricNotes`; `burlerFixEntry` accepts `color_segment` and sets `Profile.Segment`.
`colorSegmentKey` reads the optional key through `parseSegment`, so an unknown segment is refused naming `segmentcolor.Segments()`.

In `loomCLI.wire`:
- `SkipMergeReview` reads the merge record at `landingshed.MergeRecordPath(loomengine.LoomScratchDir(location))` and answers its `SkipsReview()`.
  An absent or unreadable record answers false, so the round reviews the whole merge-in diff.
  A skip happens with no agent spawned, and Merge-Bouncer's Done still records the reviewed tip, because the Bouncer's skip path calls `Approve`.
- `ApproveMergeReview` calls `landingshed.RecordMergeReviewed` over the record and reviewed paths, so every Merge-Bouncer Done, a skip included, records the merged tip.
  A review that ends without that Done records nothing.
- `MergeReview` returns `landingshed.MergeReviewNote` over the record path.
- `MergeReviewOff` is the loaded config's `SkipMergeReview`.
`envkit.FullEnv` fills the three seams with fakes, and battencli's `intentionallyNil` names them as loom-only seams batten never fills.

The entry tests gain rows:
- the merge skip seam resolving to the closure and refusing a nil one;
- the merge approve seam likewise;
- an unknown `color_segment` and an unknown `skip_seam` value refused;
- `blocking_escalates` reaching the config;
- the review row's merge note reaching its rubric;
- both Burler rows accepting `color_segment` and refusing an unknown key.

**ImpactSummary:** the Bouncer and Burler entries gain optional keys absent from every current row, `Env` gains four fields loom's wiring fills, and batten's seam-gap list grows.

**Commit:** 16: shedrecipe: merge review seams and keys on the Bouncer and Burler rows
