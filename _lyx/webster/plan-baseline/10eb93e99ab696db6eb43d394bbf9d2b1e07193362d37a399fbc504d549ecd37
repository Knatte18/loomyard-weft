# Card 32 — merged-tip-merge-base

**Create:**
- `internal/loomcli#taskMergeBase`
- `internal/loomcli#TestTaskMergeBase_FollowsTheMergedTip`

**Delete:**
- `internal/loomcli#TestReviewRange_ReworkBranchListsOnlyOwnCommitsAfterHead`

**Edit:**
- `internal/loomcli#codeSubject`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#darnTreeScope`
- `internal/loomcli#TestCodeSubject_Real`
- `internal/landingshed#DescribeInputs`
- `internal/landingshed#DarnDescribeInputs`
- `internal/landingshed#DescribeTable`
- `internal/landingshed#DarnDescribeTable`
- `internal/landingshed#newDescribeInputs`
- `internal/landingshed#TestDescribeSpec_ComposesSpec`
- `internal/landingshed#TestDarnDescribeSpec`
- `internal/landingshed#TestDescribeSpec_EmptyFieldIsError`
- `internal/shedadapters#RubricNotes`
- `internal/shedadapters#ReadRubric`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.rubricNotes`
- `internal/shedadapters#TestReadRubric`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/testkit/envkit#FullEnv`
- `contracts/stencils#rubricMarkerAllowlist`
- `contracts/stencils/loom/loom-rubric-code-review.md`
- `contracts/stencils/landing/landing-template-describe.md`
- `contracts/stencils/landing/landing-template-describe-darn.md`

**Uses:**
- `internal/loomcli#pairMergeBase`
- `internal/landingshed#ReadMergeRecord`
- `internal/landingshed#MergeRecordPath`

**Extends:**
- `internal/loomcli/doc.go`

**Intent:** Every reader of "the task's diff" takes the merge base of HEAD with the merge record's merged tip, since the local parent branch can now lag the tip a task merged (D3); with no merge record it falls back to the local parent branch, as today.
`taskMergeBase` reads the merge record from the run's scratch directory and answers `MergeBase(merged tip)` over the task pair, or `pairMergeBase`'s answer when no record exists; an unreadable record is logged at Warn and read as absent, as Merge-In reads one, so no reader errors on it and the next Merge-In or Finalize rewrites it.
The Code-Perch's subject seam `codeSubject` takes its base from it, so the settled subject and the amendment base match the review range.
The verify gate takes it as `VerifyMergeBase` on loom and darn runs: the headroom lint's base in loom's round form, and the base of both lints in darn's form; darn's tree-scope reader classifies the diff from it.
The loom describe stencil's carry-over rule, which turns a Plan-Review or Webster-Review entry's open findings into items to check by hand, names the Code-Review entry where it named Webster-Review, since the Code-Perch files its carry-over under `Code-Review`.
Describe's diff reads it: `DescribeInputs` and `DarnDescribeInputs` gain `MergeBase`, required, which `wire`'s and `wireDarn`'s Describe closures fill, and both Describe stencils name `git diff {{.merge_base}} {{.task_branch}}` in place of their merge-base example.
The Code-Review rubric's review range becomes one step over a Go-filled `review_base` marker: review `git diff <review_base>..HEAD`; the PR-Gate rework-head steps and the live-round derivation go (D5), and an empty base is the BLOCKING "range could not be determined" finding as today.
`RubricNotes` gains `ReviewBase`, `ReadRubric` fills the marker, `BouncerConfig` and `shedrecipe.Env` gain the `ReviewBase` seam the Bouncer and Burler-Review entries pass, `wire` fills it from `taskMergeBase`, and the rubric marker allowlist names the marker.
The rework-range integration test goes with the rework steps.

`TestTaskMergeBase_FollowsTheMergedTip` builds a hub fixture where the task merged a parent tip the local parent branch has not reached and asserts the base is that tip's merge base, so the task diff, the Code-Perch subject and the lints hold no parent commit; with no merge record, and with an undecodable one, it asserts the local parent branch's merge base.
`TestCodeSubject_Real` asserts the subject's base follows the merged tip, and the Describe and rubric tests pin the new marker.

`doc.go` names the merge-base reader and its fallback.

**ImpactSummary:** One merge-base reader feeds the Code-Perch subject and range, Describe, and the verify gate's lints; its consumers are the loom and darn wiring and two stencils and a rubric, all listed.
