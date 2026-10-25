# Card 12 — Beam promote refuses a task at another anchor than its parent's records

**Edit:**
- `internal/beamcli#promoteDeps`
- `internal/beamcli#promoteSteps`
- `internal/beamcli#hubPromoteDeps`
- `internal/beamcli#promoteWorld.deps`
- `internal/beamcli#TestPromote`

**Create:**
- `internal/fabricengine#ParentRecordsAnchor`
- `internal/fabricengine_test#TestParentRecordsAnchor`

**Uses:**
- `internal/lyxcwd#NamedWorktree`
- `internal/fabricengine#treeAnchor`
- `internal/hubforge#ReanchorPair`
- `internal/fabricengine#recordsBranchTip`
- `internal/fabricengine#ReadOriginFor`

**Intent:**
D10's promote refusal.
`fabricengine.ParentRecordsAnchor(l, slug)` answers the task pair's own anchor (`lyxcwd.NamedWorktree(l.HubPath, slug).AnchorRel`) and the anchor of the tree `ResetRecordsToParent` would reset it to: `treeAnchor` of the parent records branch's tip, read through `recordsBranchTip` from the local branch or origin's tracking ref exactly as the reset reads it, with the task's own anchor as the markerless fallback.
It never reads the parent's worktree, which need not be on this machine.
An absent origin record or a root pair answers the errors `ResetRecordsToParent` already names.

`promoteDeps` gains `Anchors func(slug string) (own, parent string, err error)`, which `hubPromoteDeps` fills with `ParentRecordsAnchor(prime, slug)`.
`promoteSteps` asks it right before `ResetRecords`, after the run is ended and cleared and the pair is marked, and refuses when the two differ, since resetting the records to the parent's tree would carry the parent's marker: `lyx beam promote: <slug> sits at anchor <own> and its parent's records at <parent>; way forward: run lyx fabric anchor <slug> --set <parent>, then lyx beam promote <slug>`.
The check sits there, not in `judgePromoteTask`, because the `--set` refuses a worktree whose run is unfinished or whose driver strand is stored, and promote's own earlier steps are what end and clear that run; every earlier step is skipped on the re-run when its result stands, so the re-run after the `--set` goes straight to the reset.
A read error is returned naming the slug and the re-run.

Tests:
`TestParentRecordsAnchor` (tier1, hubforge): a task forked from a beam that was then re-anchored at `sub` by `hubforge.ReanchorPair` answers own `.` and parent `sub`; with the beam's worktree removed and only its records branch left, the answer is the same; a task forked after the re-anchor answers `sub` twice.
`TestPromote` gains rows on its fake deps, whose `promoteWorld.deps` fills `Anchors` from a per-slug map defaulting to equal anchors: anchors that differ refuse naming `lyx fabric anchor <slug> --set <parent>`, with `EndRun`, `MarkBeam` and `ClearRun` run and `ResetRecords`, `SetOwners` and the dobby start not run; a re-run with equal anchors finishes the promote.

**ImpactSummary:** one new deps field with its hub filler and one refusal before the reset step, one new exported fabricengine reader; the test fake gains the field with an equal-anchor default, so every existing row promotes as today.
