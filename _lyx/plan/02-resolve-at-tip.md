# Card 2 — resolve-at-tip

**Edit:**
- `internal/mergeresolve#MergeSurface`
- `internal/mergeresolve#Resolver.Resolve` -> `func (r *Resolver) Resolve(ctx context.Context, source, tip string) (Result, error)`
- `internal/mergeresolve#fakeMergeSurface`
- `internal/mergeresolve#TestResolve_CleanMergeNoSession`
- `internal/mergeresolve#TestResolve_ConflictThenCleanScan_StagesThenConcludes`
- `internal/mergeresolve#TestResolve_ConflictStillUnresolvedAfterSession_RetriesThenAborts`
- `internal/mergeresolve#TestResolve_MergeInProgressAtEntry_AbortsBeforeNewAttempt`
- `internal/mergeresolve#TestResolve_MergeInError_StuckWithReasonNoAbort`
- `internal/mergeresolve#TestResolve_ShuttleOutcomes_MapToStuckNoConclude`
- `internal/mergeresolve#TestResolve_ShuttleOutcomes_WarnSurvivesAbortFailure`
- `internal/mergeresolve#TestResolve_ContextCancellation_SurfacedAsError`
- `internal/mergeresolve#TestResolve_AlreadyUpToDate_ResolvedNoSessionNoConclude`
- `internal/mergeresolve#TestResolve_RetryUsesDistinctReportPath`
- `internal/mergeresolve#TestResolve_ScratchDirAbsent_FirstSpecBuildCreatesIt`
- `internal/mergeresolve#TestResolve_SessionEditsTrackedFile_StagesTrackedBeforeConclude`
- `internal/mergeresolve#TestResolve_SessionLeavesUntrackedFile_AbortsStuckNamingItWithoutStaging`
- `internal/mergeresolve#TestResolve_StaleReportFromEarlierCall_IsClearedAtEntry`
- `internal/landingshed#resolver.Resolve`
- `internal/landingshed#MergeIn.Call`
- `internal/landingshed#Finalize.mergeInStep`
- `internal/landingshed#recordingResolver.Resolve` -> `func (r *recordingResolver) Resolve(ctx context.Context, source, tip string) (mergeresolve.Result, error)`
- `internal/landingshed#orderedResolver.Resolve` -> `func (r *orderedResolver) Resolve(ctx context.Context, source, tip string) (mergeresolve.Result, error)`
- `internal/landingshed#scriptedResolver.Resolve` -> `func (r *scriptedResolver) Resolve(ctx context.Context, source, tip string) (mergeresolve.Result, error)`
- `internal/landingshed#orderRecordingResolver.Resolve` -> `func (r *orderRecordingResolver) Resolve(ctx context.Context, source, tip string) (mergeresolve.Result, error)`

**Create:**
- `plan:internal/mergeresolve#fakeMergeSurface.MergeInAt` -> `func (f *fakeMergeSurface) MergeInAt(source, tip string) (fabricengine.MergeResult, error)`
- `plan:internal/mergeresolve#TestResolve_MergesTheToldTip` -> `func TestResolve_MergesTheToldTip(t *testing.T)`

**Extends:**
- `internal/mergeresolve/doc.go`

**Intent:** Carry the merged tip D2 fixes through the conflict-resolution entry seam, as an interface change only.
`MergeSurface` gains `MergeInAt(source, tip string) (fabricengine.MergeResult, error)`, which `*fabricengine.Fabric` satisfies through card 1's `Fabric.MergeInAt`.
`Resolver.Resolve` gains a `tip` argument: an empty tip calls `MergeIn(source)` as today, and a set tip calls `MergeInAt(source, tip)`; the abort, the stale-report clear and the conflict session are unchanged.
This package is outside the fabric vocabulary owner set, so the argument is named `tip`, never after a fabric side.

landingshed's `resolver` seam method takes the same argument, and its two callers pass an empty tip for now: `MergeIn.Call` and `Finalize.mergeInStep` keep merging by name until the merge-in-unlocked card passes a tip at Merge-In.
Every landingshed test fake of the seam (`recordingResolver`, `orderedResolver`, `scriptedResolver`, `orderRecordingResolver`) takes the argument and records it beside the source.

`fakeMergeSurface` gains `MergeInAt`, recording the tip, and every existing `Resolve` test passes an empty tip.
The new test runs `Resolve(ctx, "source", "abc123")` over a clean merge and asserts the fake saw `MergeInAt("source", "abc123")` and no `MergeIn`; the empty-tip path stays pinned by `TestResolve_CleanMergeNoSession`, which passes an empty tip and asserts `MergeIn`.

`doc.go` names the tip argument.

**ImpactSummary:** One interface method and one argument added; every caller passes an empty tip, so no behaviour changes until Merge-In passes one.
