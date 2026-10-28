# Card 1 — Finalize merges a moved parent in place for every recipe

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed/finalize.go`
- `internal/landingshed/doc.go`
- `internal/loomcli#landingDeps`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/testkit/envkit#LandingDeps`
- `internal/landingshed#TestFinalize_MergeJudgement`
- `internal/landingshed#TestFinalize_MergeInRequiredRetry`
- `internal/loomcli/doc.go`
- `internal/landingshed_test#TestFinalize_OverRealHub`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestReviewInputSeams`
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/recipes/pawl-recipe.yaml`
- `contracts/stencils/loom/loom-rubric-code-review.md`
- `contracts/stencils/orch/orch-template-role.md`
- `docs/overview.md`

**Delete:**
- `internal/landingshed/overlapnote.go`
- `internal/landingshed#fakeTaskMerger`
- `internal/landingshed#fakeTaskMerger.MergeIn`
- `internal/landingshed#fakeTaskMerger.MergeAbort`
- `internal/loomcli#overlapNoteInput`

**Uses:**
- `internal/landingshed#newTestDeps`
- `internal/landingshed#recordingResolver`
- `internal/landingshed#orderedResolver`
- `internal/landingshed#eventResolver`
- `internal/landingshed#newParentGuard`
- `internal/loomcli#fixerReportInput`

**Intent:**
D1 and D2: Finalize always merges the parent through its resolver, so a conflict runs the conflict session under the parent lock the row already holds, and a merge that needed the session or whose parent changes touch files the task changed is no reroute cause for any recipe.

`Finalize.mergeIntoTask` keeps only the resolver path: `Resolve(ctx, parent, "")`, an `OutcomeStuck` result returned as the Stuck reason, otherwise `parentMerge{upToDate: result.AlreadyUpToDate}`.
`parentMerge` drops `conflicted` and `conflicts`, since nothing reads them any more.
`Finalize.mergeParent` drops the aborted-conflict reroute, the overlap-note write and the large-merge `switch`: after the clean-tree check it judges the merge through `judgeParentMerge`, writes the merge record with `Overlap` as now, and returns done false so `mergeAndGate` goes on to `gateAndFix`, whose failing `finalize` gate runs the fixer in finalize mode.
`Finalize.rerouteReason`, `Finalize.reroute` and `Finalize.mergedTipRead` stay, read by `keepFixerCommits` and `judgeParentMerge`.
`finalize.go` is listed as a whole file because the change touches `Finalize`, `NewFinalize`, `Finalize.Call`, `Finalize.mergeParent`, `Finalize.mergeIntoTask` and `parentMerge` and deletes `taskMerger` with its `var _ taskMerger` assertion, which no member glyph spans.

`taskMerger`, its compile-time assertion and `Finalize`'s `taskMerger` field are deleted, since no recipe aborts a conflict.
`conflictSessions` and `Finalize`'s `sessions` field are deleted too: they counted sessions only to tell a conflicted merge, which nothing reads now.
`NewFinalize` hands `deps.Shuttle` to `mergeresolve.New` as told, so a nil shuttle is still refused by the resolver's constructor; the `shuttleengine` import goes if no other use remains.
`Deps.ConflictReroutes` and `Deps.LargeMergeLands` are deleted, with `landingDeps`'s two settings and their comments, and `envkit.LandingDeps`'s `ConflictReroutes: true`.
`overlapnote.go` is deleted (`WriteOverlapNote`, `OverlapNotePath`, `overlapNoteFile`), with `overlapNoteInput`; `loomCLI.wireDarn` drops its `c.env.ReviewInput` override, so a darn run keeps the shared wiring's `fixerReportInput` seam, which answers not found since darn has no Parent-Align.
`Finalize.Call`'s doc and its retry comment say the retry after a merge-in-required error merges, gates and lands a code-side parent commit instead of rerouting.

Bound (a guard removed): what now lands with no review is the conflict session's resolution against the parent's newer work and a textually clean merge of parent changes to files the task also changed, in loom and darn as already in pawl.
The `finalize` gate over the merged tree bounds both; a failing gate runs the fixer, whose attempts and size and kind caps bound its change, and a spent cap or over-cap change still reroutes through `keepFixerCommits` when `RerouteReviewed` answers true and halts with `LandingStopPrefix` otherwise.
The conflict session runs while the parent lock is held, so sibling landings onto that parent wait, bounded by the lock's existing owner, pause and release rules.

Tests:
- `TestFinalize_MergeJudgement` loses the `loom`, `largeLands`, `merge`, `session`, `wantReroute`, `wantInReason`, `wantAborted` and `wantNote` fields and `fakeTaskMerger`, and gains an `upToDate bool` field: the row's `eventResolver` result is resolved with `AlreadyUpToDate` set to it, and its `onResolve` moves the head to `t2` only when it is false.
  Its struct-literal `Finalize` carries no `taskMerger` or `sessions` and gains `guard: newParentGuard(deps)`, and the `eventResolver`'s `lockHeld` reads whether `deps.Lock.Store` names the run for the parent branch, so every row asserts `heldAtMerge`: the merge-in, and with it any conflict session the resolver starts, runs under the parent lock.
  Every row expects Done after one verify call and the merge record as now: an up-to-date merge; an unreadable record measured from the task's merge base, which lands with `x.go` in the record's overlap; an overlapping parent change, which lands with `b.go` in the record's overlap and the merge commit kept, the large merge that used to reroute; a non-overlapping change; and a run with no record measured from the merge base.
  The former conflict-session rows go with the `sessions` counter, since production no longer tells a merge that needed the session from one that did not; `TestFinalize_OverRealHub` drives the real conflict session.
  Each row also asserts no `overlap-note.md` exists under `deps.ScratchDir` after the call.
  The file's header comment drops the fake task merger.
- `TestFinalize_MergeInRequiredRetry` gains a row for a code-side parent commit that arrives between the passes: `MergeBase` answers `b0`; `ReadParentTips` answers `Local` and `Upstream` both `p2` with `Relation: fabricengine.ParentEqual` until the first parent-side merge call, and `p3` with the same relation from then on; `ChangedPaths` answers `x.go` for every range and records each `from..to`.
  The first pass reads an overlap since `testParentTip`, the tip `newTestDeps`'s merge record names, and the retry reads a fresh overlap since `p2`, the merge record's tip: the retry lands, with outcome Done, order `merge-in,records,merge,merge-in,records,merge,mark,push`, two parent-side merge calls, Reroute false and a `p2..p3` call among the `ChangedPaths` calls.
- A large merge whose gate fails reaches the fixer through the same `gateAndFix` as any other merge, which reads nothing of the overlap: `TestFinalize_MergeJudgement`'s overlap row pins that a large merge reaches the gate, and `TestFinalize_GateAndFixer`'s existing subtests pin the fixer, its caps, the reroute and the halt over a failing gate.
- `TestFinalize_OverRealHub`'s first step expects one `Finalize` call to resolve the conflict with one conflict session and land Done, and drops the reroute and second call; its doc comment says so.
- `TestLandingDeps_EveryFieldPopulated` drops the `ConflictReroutes` check and the `LargeMergeLands` entry of its unfilled map; `TestWire_BeamRecipe` drops the `wantLargeMerge` and `wantConflictReroutes` columns and their checks, and its subtest name drops "let a large merge land".
- `TestWire_DarnRecipe` drops the overlap-note write and asserts the darn `ReviewInput` is told and answers not found; `TestReviewInputSeams` drops its `OverlapNote` row.

Docs: `internal/landingshed/doc.go`'s Finalize merge paragraph says Finalize merges through its resolver for every recipe, a conflict runs the conflict session under the lock, and a large merge writes the merge record and goes on to the gate, with the bound above.
The loom and darn recipe header and Finalize row comments say the `on_reroute` edge carries only a spent or over-cap fixer change.
The pawl recipe header gives its missing `on_reroute` edge the reason that remains: a spent or over-cap fixer has no review to return to, so it halts for the orch, while a conflict or a large merge lands through the gate in every recipe.
`internal/loomcli/doc.go`'s pawl paragraph drops "lets a large merge go on to the gate in place of a reroute", which every recipe now does, and keeps the halt for a spent or over-cap fixer.
The Code-Review rubric's review-input paragraph names only the merge fixer's Parent-Align report.
The orch role stencil's `Landing halts` says a moved parent, a conflict or a large merge no longer reroutes, and its parent-lock bullet says the released run merges the moved parent at its next Finalize.
`docs/overview.md`'s Finalize line says Finalize merges the parent in place, resolving a conflict under the lock, and gates the merged tree with no reroute on a conflict or a large merge.

**ImpactSummary:** Finalize's merge path collapses to the resolver within `internal/landingshed`; the two removed `Deps` fields and the overlap note reach only `internal/loomcli`'s landing and darn wiring, `envkit.LandingDeps` and their tests, with no exported signature change elsewhere.
