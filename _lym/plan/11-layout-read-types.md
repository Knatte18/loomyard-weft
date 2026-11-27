# Card 11 — layout-read-types

**Move:**
- `internal/fabricengine#CheckoutState`
- `internal/fabricengine#CodeWorktree`
- `internal/fabricengine#WorktreeEntry`
- `internal/fabricengine#WorktreeEvent`
- `internal/fabricengine#AnchoredFileVersion`
- `internal/fabricengine#ConfigChanges`
- `internal/fabricengine#ConfigFileContents`
- `internal/fabricengine#ChangeSide`
- `internal/fabricengine#SideWarp`
- `internal/fabricengine#SideWeft`
- `internal/fabricengine#ChangeEntry`
- `internal/fabricengine#DiffResult`
- `internal/fabricengine#HealthCause`
- `internal/fabricengine#CauseBranchMismatch`
- `internal/fabricengine#CauseConfigLoadFailed`
- `internal/fabricengine#CauseJunctionMissing`
- `internal/fabricengine#CauseNotAJunction`
- `internal/fabricengine#CauseJunctionPointsElsewhere`
- `internal/fabricengine#HealthReason`
- `internal/fabricengine#LineChange`
- `internal/fabricengine#DiffSize`
- `internal/fabricengine#FileDiff`
- `internal/fabricengine#TaskPatch`
- `internal/fabricengine#MoveRecord`
- `internal/fabricengine#Origin`
- `internal/fabricengine#PairParts`
- `internal/fabricengine#PairParts.Any`
- `internal/fabricengine#RecordsTag`
- `internal/fabricengine#Trailer`
- `internal/fabricengine#TrailerCommit`
- `internal/fabricengine#PollutionEntry`

**Edit:**
- `cmd/lym#destructiveGuardReadOnlyResultTypes`
- `internal/fabricengine/doc.go`

**Intent:** Move the read verbs' plain result types into `internal/fabriclayout`'s `readtypes.go` (D1, D7), and retarget every caller:
the worktree, checkout and anchored-file reads, the config-change and diff reads, the health reason, the line and patch sizes, the move and origin records, the pair parts, the records tag and the trailer values.
Each is plain data; `PairParts.Any` is pure and moves with its type.
The same-named engine methods `Fabric.LineChange` and `Topology.PairParts` stay, returning the layout types.
`WorktreeSeam`, an interface the engine installs, stays; its `WorktreeEvent` argument moves.

`cmd/lym#destructiveGuardReadOnlyResultTypes`' `DiffResult` row names `internal/fabriclayout/readtypes.go`, so the guard still checks it embeds no record.
`doc.go` names the `fabriclayout` home of `DiffResult.NoWeftCorrespondence` and `HealthReason` where it names them as this package's own; its rules are unchanged.
Every reference to a moved name, in the engine and in every other production and test file, becomes `fabriclayout.<Name>`, imports following as card 3 says.
The read verbs' tests stay in the engine: each asserts a read the engine runs.
No behavior changes (D5).

**ImpactSummary:** Thirty-one read result names move with mechanical retargets across their readers outside fabric, `internal/hubgeom` and the CLI packages chief among them; one guard row follows.

**Commit:** `11: layout-read-types`
