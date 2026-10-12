# Card 10 — landing-merge-record

**Create:**
- `internal/landingshed#MergeRecord`
- `internal/landingshed#Overlap`
- `internal/landingshed#MergeRecordPath`
- `internal/landingshed#MergeReviewedPath`
- `internal/landingshed#ReadMergeRecord`
- `internal/landingshed#WriteMergeRecord`
- `internal/landingshed#ReadReviewedTip`
- `internal/landingshed#RecordMergeReviewed`
- `internal/landingshed#computeOverlap`
- `internal/landingshed#MergeRecord.SkipsReview`
- `internal/landingshed#MergeReviewNote`
- `internal/landingshed/mergerecord_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the Go-owned merge record Merge-In writes and the merge review reads (D1, D6), with no producer change yet.

`MergeRecord` holds, as YAML:
- `parent_branch`;
- `merged_tip`, the parent tip the merge brought in;
- `reviewed_tip`, the parent tip the last finished merge review reached, the fork point before any;
- `task_base`, the parent tip last merged into the task before this merge, the fork point before any merge-in;
- `task_head`, the task head before this merge;
- `merge_commit`, the merge commit, empty for an up-to-date merge;
- `up_to_date`;
- `overlap`, an `Overlap` of `files`, `packages` and `pattern` path lists.

`MergeRecordPath(scratchDir)` and `MergeReviewedPath(scratchDir)` join fixed file names onto the told run scratch directory, so the record stays under `.lyx` (`PATTERN-durable-vs-ephemeral-state`).
`ReadMergeRecord` reports found false for an absent file and an error for an undecodable one; `WriteMergeRecord` writes atomically.
`ReadReviewedTip` reads the one-line reviewed tip.
`RecordMergeReviewed(recordPath, reviewedPath)` copies the merge record's `merged_tip` into the reviewed file atomically; a missing merge record writes nothing and returns nil, so the next range still starts at the earlier reviewed tip.

`computeOverlap(parentPaths, taskPaths)` sorts and returns:
- `files`, the paths changed on both sides;
- `packages`, the directories in which both sides changed a `.go` file;
- `pattern`, the parent-side paths that are `PATTERN.md` or under `pattern/`.

`SkipsReview()` is true for an up-to-date merge and for a record whose three lists are all empty; Bound: it skips only when the git-computed sides share no file, no Go package and no PATTERN change.

`MergeReviewNote(recordPath)` renders the merge review's subject as the markdown note the rubric's `merge_review` marker carries:
- the parent range `<reviewed_tip>..<merged_tip>`;
- the task range `<task_base>..<task_head>`;
- the merge commit;
- the overlapping files, packages and pattern files.
For a missing or unreadable record it renders a note telling the reviewer to review the whole merge-in diff of the latest merge of the parent into the task, which is D6's fallback.

`mergerecord_test.go` covers, in tables:
- the record round-trip and an undecodable file;
- `computeOverlap` for a shared file, a shared package directory with different files, a parent change to `PATTERN.md` and to a file under `pattern/`, and disjoint sides;
- `SkipsReview` for up to date, empty overlap and each non-empty list;
- `RecordMergeReviewed` with and without a merge record;
- `MergeReviewNote` for a record and for a missing one.

**ImpactSummary:** new record helpers in landingshed with no caller yet.

**Commit:** 10: landing: the merge record, its overlap and the review note
