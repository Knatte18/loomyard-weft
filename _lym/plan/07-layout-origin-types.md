# Card 7 — layout-origin-types

**Move:**
- `internal/fabricengine#cleanupRerun`
- `internal/fabricengine#ReconcileScope`
- `internal/fabricengine#EndedTeardown`
- `internal/fabricengine#BusyOnDriver`
- `internal/fabricengine#BusyOnLock`
- `internal/fabricengine#TeardownReport`
- `internal/fabricengine#BranchState`
- `internal/fabricengine#BranchLevel`
- `internal/fabricengine#BranchFastForward`
- `internal/fabricengine#BranchAhead`
- `internal/fabricengine#BranchRewound`
- `internal/fabricengine#BranchRewritten`
- `internal/fabricengine#BranchDiverged`
- `internal/fabricengine#BranchNeedsReplay`
- `internal/fabricengine#BranchGone`
- `internal/fabricengine#BranchDirty`
- `internal/fabricengine#BranchBusy`
- `internal/fabricengine#BranchLive`
- `internal/fabricengine#BranchAnchorStopped`
- `internal/fabricengine#BranchState.Stops`
- `internal/fabricengine#BranchOutcome`
- `internal/fabricengine#BranchOutcome.rerunsCleanup`
- `internal/fabricengine#EntryState`
- `internal/fabricengine#EntryActive`
- `internal/fabricengine#EntryEnded`
- `internal/fabricengine#EntryStopped`
- `internal/fabricengine#EntryOwnedElsewhere`
- `internal/fabricengine#EntryOutcome`
- `internal/fabricengine#OriginReport`
- `internal/fabricengine#OriginReport.Summary`
- `internal/fabricengine#ParentRelation`
- `internal/fabricengine#ParentEqual`
- `internal/fabricengine#ParentBehind`
- `internal/fabricengine#ParentAhead`
- `internal/fabricengine#ParentDiverged`
- `internal/fabricengine#ParentNoUpstream`
- `internal/fabricengine#ParentTips`
- `internal/fabricengine#FastForwardRefusal`
- `internal/fabricengine#FastForwardDirty`
- `internal/fabricengine#FastForwardMergeInProgress`
- `internal/fabricengine#FastForwardNotFastForward`
- `internal/fabricengine#FastForwardResult`
- `internal/fabricengine#CodeTipReading`
- `internal/fabricengine#PendingTeardown`
- `internal/fabricengine#HandOverMark`
- `internal/fabricengine#LedgerRun`
- `internal/fabricengine#ReplayFile`
- `internal/fabricengine#ReplayRequest`
- `internal/fabricengine#ReplayFinding`
- `internal/fabricengine#ReplayResult`

**Create:**
- `internal/fabriclayout#EntryOutcome.AdoptTarget`
- `internal/fabricengine#branchStopWayForward`
- `internal/fabricengine#branchAnchorStopWayForward`
- `internal/fabricengine#branchDirtyWayForward`

**Delete:**
- `internal/fabricengine#EntryOutcome.adoptSubject`
- `internal/fabricengine#BranchOutcome.stopWayForward`
- `internal/fabricengine#BranchOutcome.anchorStopWayForward`
- `internal/fabricengine#BranchOutcome.dirtyWayForward`

**Edit:**
- `internal/fabricengine#judgeAgainstOrigin`
- `internal/fabricengine#originMover.move`
- `internal/fabricengine#classifyEntries`
- `internal/fabricengine#StopWayForwardForTest`
- `internal/fabricengine/doc.go`

**Intent:** Move the origin reconcile's, the parent tips', the teardown's, the hand-over's and the records replay's plain result types into `internal/fabriclayout`'s `origintypes.go` (D1, D7), and retarget every caller.
They are plain data: strings, bools, slices, `time.Time`, `error` and the embedded `fabriclayout.MutationRecord`.
The seam and option types beside them stay in the engine, since each holds function fields:
`ReconcileOptions`, `OriginSeams`, `OwnerReader`, `BoltReplayer`, `RunLedger`.

The moved methods are pure and move with their types: `BranchState.Stops`, `OriginReport.Summary` and the `rerunsCleanup` and `cleanupRerun` it reads.
Two of today's methods cannot move as they are:
- `EntryOutcome.adoptSubject` is read by `Summary` in the layout and by the engine's `classifyEntries`, so it becomes the exported `EntryOutcome.AdoptTarget`, same body; `Summary` and `classifyEntries` call it.
  `AdoptSubject` is already a field name, hence `AdoptTarget`.
- `BranchOutcome.stopWayForward`, `anchorStopWayForward` and `dirtyWayForward` read disk and `lymcwd` (`refuseEmptyOwnMarker`, `lymVerbDir`) and build failed-push lines, so they stay in the engine as functions over a `fabriclayout.BranchOutcome`, bodies unchanged.
  `judgeAgainstOrigin`, `originMover.move` and `StopWayForwardForTest` call `branchStopWayForward`.

Every reference to a moved name, in the engine and in every other production and test file, becomes `fabriclayout.<Name>`, imports following as card 3 says.
`doc.go`'s origin reconcile section names `fabriclayout`'s `OriginReport`, `BranchOutcome`, `EntryOutcome` and `ReconcileScope` where it names them as this package's own; its rules are unchanged.
The tests of these types stay where they are: each one asserts a verdict, a teardown or a replay through the engine, which stays.
No behavior changes (D5): every report, way forward and summary line is the same text as before.

**ImpactSummary:** Fifty plain result names move with mechanical retargets outside; inside the engine four method-to-function or method rename changes touch four callers in the reconcile files and one test accessor.

**Commit:** `7: layout-origin-types`
