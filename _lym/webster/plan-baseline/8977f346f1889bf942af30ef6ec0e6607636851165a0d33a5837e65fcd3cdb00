# Card 15 — drift-judge-seat

**Create:**
- `internal/driftengine#JudgeRequest`
- `internal/driftengine#WriteJudgeRequest`
- `internal/driftengine#JudgeInputs`
- `internal/driftengine#JudgeTable`
- `internal/driftengine#NewJudgementGate`
- `internal/driftengine#JudgeSnapshot`
- `internal/driftengine#TakeJudgeSnapshot`
- `internal/driftengine#ReadJudgeSnapshot`
- `internal/driftengine#JudgeSnapshot.Changes`
- `internal/driftengine#TestJudgeSnapshot`
- `internal/driftengine#TestJudgementGate`
- `internal/lymdirs#DriftJudgeDirName`
- `contracts/stencils#DriftTemplateJudge`
- `contracts/stencils/drift/drift-template-judge.md`

**Extends:**
- `internal/driftengine#Deps`
- `internal/driftengine#Paths`
- `internal/driftengine/doc.go`
- `internal/lymdirs/doc.go`
- `internal/shedrecipe/doc.go`

**Edit:**
- `internal/driftengine#NewPaths`
- `internal/driftengine#TestSeatTables`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/shedrecipe#TestResolveGateSpec_UnknownNameListsTheLegalValues`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `internal/loomcli#driftDeps`
- `internal/loomcli#driftTables`
- `internal/testkit/envkit#driftDeps`
- `internal/battencli#intentionallyNil`

**Uses:**
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/fabricengine#CheckoutStates`
- `internal/fabricengine#WeftPathsChangedSince`
- `internal/shedrun#RunDirRel`
- `internal/loomengine#LoomFrictionDirRel`
- `internal/fabricengine#HubScratchDir`
- `internal/driftengine#ParseJudgement`
- `internal/driftengine#JudgementWant`
- `internal/driftengine#BaselineDigest`
- `internal/driftengine#copyFile`

**Intent:** The judge's seat, its `judgement` entry and the snapshot that guards what the judge session may change (D9); no row spawns it yet.

- `Paths` gains `JudgeRequest`, `.lym/drift/judge-request.json`: the `JudgeRequest` Drift-Verify writes before each spawn, with `SpawnID`, `Mode`, the cells as id, case, identity, kind, old, new, covering `why`, whether that entry is an open range and, for an `added case` cell, the absolute path of the case's file in the current capture's scratch directory and, for a `removed case` cell, that of its file in the baseline directory, so the judge reads the new or the old file (D9); the open ranges covering a judged cell; or under `command` the compare's log; and the expectation and amendments as text.
- `Deps` gains `JudgeTable func(request string) (seatengine.Table, error)`, `Checkouts func() (code, records fabricengine.CheckoutState, err error)`, `WeftChangedSince func(rev string) ([]string, error)`, `JudgeSnapshotDir string` and `LoomWeftDirs []string`, the weft paths loom itself writes during a row, relative to the weft worktree's root, each a directory or, ending in `-`, a path prefix.
  `loomcli`'s `driftDeps` wires `JudgeTable`, `Checkouts` over `CheckoutStates`, `WeftChangedSince` over `WeftPathsChangedSince`, `LoomWeftDirs` as the run's shed directory, `shedrun.RunDirRel` of the run, loom's friction directory, `loomengine.LoomFrictionDirRel`, and the prefix of its timestamped archive siblings, `LoomFrictionDirRel` followed by `-`, the directories `loomengine.IsLoomFrictionArchive` accepts and `preflight.runStateExemptRels` exempts, each joined under the location's `AnchorRel` and slash-separated, and `JudgeSnapshotDir` as `<HubScratchDir>/drift-judge/<slug>`, under the hub's board-level `.lym` beside the run process records, its name the `lymdirs` constant (`PATTERN-durable-vs-ephemeral-state`); `envkit` fills fakes and `battencli`'s `intentionallyNil` lists the three func seams.
- `JudgeTable` builds one lead seat on the new `drift-template-judge` stencil, role `drift-judge`, the review segment, model `cfg.JudgeModel`, interactive under `cfg.JudgeInteractive`, where its mode text has it present each cell or case to the operator and record the operator's ruling; skill `scribe:prose`; its one output `judgement.yaml`; values the request path, the judgement path and the task's subject reading.
- `NewJudgementGate` reads the request and parses `judgement.yaml` with `ParseJudgement` against the `JudgementWant` it implies, failing with the parser's message; a missing file fails asking for it.
- `TakeJudgeSnapshot` records under `JudgeSnapshotDir`: both checkouts' `CheckoutState`, a content hash of every changed tracked path and a copy, through `copyFile`, of every untracked path beside it, and the path and hash of every file in the current capture's scratch directory.
  `ReadJudgeSnapshot` reads it back, found false when it is gone.
  `Changes` compares both checkouts and the scratch directory with the snapshot and names each moved HEAD, each changed, added or deleted tracked or untracked path outside the exempt ones, and each changed scratch file; a scratch directory that is gone is left out (D9).
  The exempt paths are `judgement.yaml`, the file the judge writes, and those loom itself writes on the weft during a row: every path under a `LoomWeftDirs` directory or starting with a `LoomWeftDirs` prefix, that is the run's shed directory, which holds the `status.json` loom commits at each row transition, the friction directory, which takes the halt and crash-resume notes loom writes and commits at a halt and a `lym loom start` or `step`, and the friction archive siblings.
  Bound: an edit the judge session makes under the exempt run directory, friction directory or friction archive siblings goes unseen by the guard; those paths are loom's own run bookkeeping and hold no code, no expectation and neither `scope.json` nor `judged.json`; afterwards loom reads the run directory's `status.json` and settle records to drive, resume and settle the run, and the stop report reads the friction notes as the run's halt notes, so a judge edit there can mislead loom's run state, its settling or a stop report.
  It decides no containment: containment reads its cells, rulings and coverage from the records, the sealed expectation, the baseline and the current capture, and from the run directory only the status row, to tell whether the run is past Drift-Expect, which loom rewrites and commits at the row's own transition, so a later check reads loom's status row and never one the judge left.
  A moved weft HEAD counts as a change only when `WeftChangedSince` of the snapshot's weft head names a path outside the exempt ones, and the weft's `Changed` and `Untracked` lists drop the exempt paths before the compare, so loom's own commits are never read as the judge's edits (D9 guards what the session made).
  A weft path is judged by content: one that `WeftChangedSince`, `Changed` or `Untracked` names is no change when it was already uncommitted at the snapshot and its content now equals the hash or copy the snapshot recorded for it, or when it was an uncommitted deletion at the snapshot and is still absent, so a later loom commit of content that was already there, such as the reviews or loom durable directory a transition's status commit stages, is never read as the judge's edit.
  Its record is written through `internal/state`.
- The stencil rules each cell or case against its `why` per D9, writes the spawn id it was handed into `judgement.yaml`, lists every open range covering a judged cell with what it checked under it, and edits nothing else; it carries the directive markers, and `roleOpeningStencils` maps `drift-judge` to it.
- `resolvePlainGateClosure` resolves `judgement` to `NewJudgementGate` over `driftDepsFrom(env)`.

`TestJudgeSnapshot` (tier0, fake `Checkouts` over `t.TempDir` trees, `t.Parallel`) covers: a modified tracked file, a modified or deleted untracked file, an added untracked file, a moved HEAD on either side and a changed scratch file are each named; a change present before the snapshot is not; `judgement.yaml` is not; a weft commit after the snapshot changing only the run's `status.json` is no change, nor is one adding a friction note, or such a note left untracked, while a commit also changing another weft path names that path; a file under the reviews directory untracked at the snapshot and committed afterwards with the same content is no change, while a changed content or a new path there is named; a tracked file there deleted and uncommitted at the snapshot whose deletion is committed afterwards is no change; a path under a friction archive sibling is no change; a deleted scratch directory is left out; a gone snapshot reads found false.
`TestJudgementGate` (tier0) covers the entry passing a valid file and failing on a missing file and on a file carrying another spawn id.
`TestSeatTables` gains the judge table's rows, autonomous and interactive; the resolver tests and the stencil marker test gain the new names.

**ImpactSummary:** Adds the judge seat, gate name, stencil, snapshot and five Deps fields wired in loomcli; one new lymdirs constant; nothing runs it until Drift-Verify does.
