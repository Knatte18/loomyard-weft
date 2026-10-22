# Card 1 — guard-error-carries-dirt

**Create:**
- `internal/fabricengine#codeDirtyPaths`
- `internal/fabricengine#MergeGuardError.has`
- `internal/fabricengine#MergeGuardError.withDirt`
- `internal/fabricengine#MergeGuardError.AlreadyInProgress`
- `internal/fabricengine#MergeGuardError.NotSynced`
- `internal/fabricengine#MergeGuardError.DetachedHead`

**Edit:**
- `internal/fabricengine#MergeGuardError`
- `internal/fabricengine#MergeGuardError.WorktreeDirty`
- `internal/fabricengine#pairDirtyReason` -> `func pairDirtyReason(f *Fabric) ([]string, []string, error)`
- `internal/fabricengine#Fabric.Merge`
- `internal/fabricengine#Fabric.MergeIn`
- `internal/fabricengine#Fabric.recheckMergePreconditionsUnderLock`
- `internal/fabricengine#TestMergeGuardError_WorktreeDirty`
- `internal/fabricengine#TestMergeVocabulary_ErrorsAreSideFree`
- `internal/fabricengine_test#TestMerge_DirtyTargetHalts`
- `internal/fabricengine_test#TestMergeIn_DirtyWarpRefuses`
- `internal/fabricengine_test#TestMergeIn_PairTurningDirtyWhileWaitingForLock_RefusesPreservingDirt`
- `internal/fabricengine_test#TestMerge_PairTurningDirtyWhileWaitingForLock_RefusesPreservingDirt`
- `internal/fabricengine_test#TestWeftGuards_DetachedWeftDoesNotRefuseWarpDetachedStillDoes`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#trackedDirtyPaths`
- `internal/fabricengine#newMergeGuardError`

**Intent:**
D2's guard half: the dirty guard decides on the path list it reads, and the guard error carries that list and the checkout it read, outside `Error()`.

`codeDirtyPaths` reads `trackedDirtyPaths(f.warpPath)` once and returns each path relative to `f.warpPath`, slash-separated (`filepath.Rel` then `filepath.ToSlash`) and sorted, so a path holding a space comes back verbatim from the `-z` read.
`pairDirtyReason` now calls `codeDirtyPaths` in place of `worktreeDirty(scopeTracked, f.warpPath)` and decides on the list: it returns `[]string{mergeReasonWorktreeDirty}` and the list when the list is not empty, and nil, nil otherwise, so the error never says dirty with an empty list.
Its error wrap (`fabricengine: check checkout dirtiness: %w`) stays.

`MergeGuardError` gains two exported fields, `Checkout string` (the absolute path of the code checkout the dirty guard read) and `DirtyPaths []string` (the list above).
Both are set only when the error carries the dirty-worktree reason, and `Error()` stays the fixed, path-free string the closed reason set requires; the type's godoc and the file's header comment say the fields carry the detail, as `ErrMergeInRequired.Source` does.
`withDirt` sets both fields when `paths` is not empty and returns `e`, so a site writes `newMergeGuardError(reasons).withDirt(f.warpPath, dirtyPaths)`.
Every site that calls the dirty guard fills them this way: the aggregated guard stage of `Fabric.Merge` and of `Fabric.MergeIn`, and the dirty arm of `recheckMergePreconditionsUnderLock`.

`has` reports whether the reason list holds a reason; `WorktreeDirty` is rewritten over it, and three accessors join it so a caller outside the package tells the reasons Finalize words apart without a literal: `AlreadyInProgress` (`merge already in progress`), `NotSynced` (`branch not synced to upstream`) and `DetachedHead` (`checkout is not on a branch`).

`internal/fabricengine/doc.go`'s merge-guard passage, where it names `pairDirtyReason`'s scope, says the dirty guard decides on the code checkout's worktree-relative, sorted tracked dirty paths and that a guard error carrying the dirty reason carries that checkout and those paths in `Checkout` and `DirtyPaths`, outside `Error()`.

Tests:
- `TestMergeGuardError_WorktreeDirty` grows a column per accessor, so each row asserts all four accessors over its reasons; a row carrying only the detached reason and one carrying only the not-synced reason are added, so each accessor has a true and a false row.
- `TestMergeVocabulary_ErrorsAreSideFree` builds a guard error over every pinned reason with `withDirt("/hub/parent", []string{"a b.go"})` and asserts its `Error()` holds neither `/hub/parent` nor `a b.go`.
- `TestWeftGuards_DetachedWeftDoesNotRefuseWarpDetachedStillDoes`' detached-warp refusal, a guard error with no dirty reason, asserts `Checkout` and `DirtyPaths` are empty, so the fields are set only beside the dirty reason.
- The acceptance criterion that a dirty-reason error always carries a non-empty list, from each site: `TestMerge_DirtyTargetHalts` (`Merge`'s guard stage) asserts `Checkout` equals the target's code checkout and `DirtyPaths` equals `[]string{"tracked.txt"}`;
  `TestMergeIn_DirtyWarpRefuses` (`MergeIn`'s guard stage) asserts the prime worktree and `[]string{"dirty.txt"}`;
  `TestMergeIn_PairTurningDirtyWhileWaitingForLock_RefusesPreservingDirt` and `TestMerge_PairTurningDirtyWhileWaitingForLock_RefusesPreservingDirt` (the under-lock re-check, reached only after the pre-lock guard passed on a clean pair) assert the prime worktree and `[]string{"overlap.txt"}`.
  Each compares `Checkout` after `filepath.EvalSymlinks` on both sides, since a temp root may be a symlink.

**ImpactSummary:** the dirty guard and its three in-package call sites inside `fabricengine`, plus four additive accessor and field members on `MergeGuardError`; `Error()` and the reason set are unchanged, so `fabriccli`'s envelope and every outside caller keep their text.
