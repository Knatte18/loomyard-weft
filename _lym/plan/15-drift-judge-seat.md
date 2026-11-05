# Card 15 — drift-judge-seat

**Create:**
- `plan:internal/driftengine#JudgeRequest` -> `type JudgeRequest struct`
- `plan:internal/driftengine#WriteJudgeRequest` -> `func WriteJudgeRequest(p Paths, req JudgeRequest) error`
- `plan:internal/driftengine#JudgeInputs` -> `type JudgeInputs struct`
- `plan:internal/driftengine#JudgeTable` -> `func JudgeTable(in JudgeInputs, cfg Config, reg modelspec.Registry) (seatengine.Table, error)`
- `plan:internal/driftengine#NewJudgementGate` -> `func NewJudgementGate(d Deps) shuttleengine.Gate`
- `plan:internal/driftengine#JudgeSnapshot` -> `type JudgeSnapshot struct`
- `plan:internal/driftengine#TakeJudgeSnapshot` -> `func TakeJudgeSnapshot(d Deps, scratchDir string) (JudgeSnapshot, error)`
- `plan:internal/driftengine#ReadJudgeSnapshot` -> `func ReadJudgeSnapshot(d Deps) (JudgeSnapshot, bool, error)`
- `plan:internal/driftengine#JudgeSnapshot.Changes` -> `func (s JudgeSnapshot) Changes(d Deps) ([]string, error)`
- `plan:internal/driftengine#TestJudgeSnapshot` -> `func TestJudgeSnapshot(t *testing.T)`
- `plan:internal/driftengine#TestJudgementGate` -> `func TestJudgementGate(t *testing.T)`
- `plan:internal/lymdirs#DriftJudgeDirName` -> `const DriftJudgeDirName = "drift-judge"`
- `plan:contracts/stencils#DriftTemplateJudge` -> `var DriftTemplateJudge string`
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
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `plan:internal/fabricengine#CheckoutStates`
- `internal/fabricengine#HubScratchDir`
- `plan:internal/driftengine#ParseJudgement`
- `plan:internal/driftengine#JudgementWant`
- `plan:internal/driftengine#BaselineDigest`
- `internal/driftengine#copyFile`

**Intent:** The judge's seat, its `judgement` entry and the snapshot that guards what the judge session may change (D9); no row spawns it yet.

- `Paths` gains `JudgeRequest`, `.lym/drift/judge-request.json`: the `JudgeRequest` Drift-Verify writes before each spawn, with `SpawnID`, `Mode`, the cells as id, case, identity, kind, old, new, covering `why`, whether that entry is an open range and, for an `added case` cell, the absolute path of the case's file in the current capture's scratch directory and, for a `removed case` cell, that of its file in the baseline directory, so the judge reads the new or the old file (D9); the open ranges covering a judged cell; or under `command` the compare's log; and the expectation and amendments as text.
- `Deps` gains `JudgeTable func(request string) (seatengine.Table, error)`, `Checkouts func() (code, records fabricengine.CheckoutState, err error)` and `JudgeSnapshotDir string`.
  `loomcli`'s `driftDeps` wires `JudgeTable`, `Checkouts` over `CheckoutStates`, and `JudgeSnapshotDir` as `<HubScratchDir>/drift-judge/<slug>`, under the hub's board-level `.lym` beside the run process records, its name the `lymdirs` constant (`PATTERN-durable-vs-ephemeral-state`); `envkit` fills fakes and `battencli`'s `intentionallyNil` lists the two func seams.
- `JudgeTable` builds one lead seat on the new `drift-template-judge` stencil, role `drift-judge`, the review segment, model `cfg.JudgeModel`, interactive under `cfg.JudgeInteractive`, where its mode text has it present each cell or case to the operator and record the operator's ruling; skill `scribe:prose`; its one output `judgement.yaml`; values the request path, the judgement path and the task's subject reading.
- `NewJudgementGate` reads the request and parses `judgement.yaml` with `ParseJudgement` against the `JudgementWant` it implies, failing with the parser's message; a missing file fails asking for it.
- `TakeJudgeSnapshot` records under `JudgeSnapshotDir`: both checkouts' `CheckoutState`, a content hash of every changed tracked path and a copy, through `copyFile`, of every untracked path beside it, and the path and hash of every file in the current capture's scratch directory.
  `ReadJudgeSnapshot` reads it back, found false when it is gone.
  `Changes` compares both checkouts and the scratch directory with the snapshot and names each moved HEAD, each changed, added or deleted tracked or untracked path other than `judgement.yaml`, and each changed scratch file; a scratch directory that is gone is left out (D9).
  Its record is written through `internal/state`.
- The stencil rules each cell or case against its `why` per D9, writes the spawn id it was handed into `judgement.yaml`, lists every open range covering a judged cell with what it checked under it, and edits nothing else; it carries the directive markers, and `roleOpeningStencils` maps `drift-judge` to it.
- `resolvePlainGateClosure` resolves `judgement` to `NewJudgementGate` over `driftDepsFrom(env)`.

`TestJudgeSnapshot` (tier0, fake `Checkouts` over `t.TempDir` trees, `t.Parallel`) covers: a modified tracked file, a modified or deleted untracked file, an added untracked file, a moved HEAD on either side and a changed scratch file are each named; a change present before the snapshot is not; `judgement.yaml` is not; a deleted scratch directory is left out; a gone snapshot reads found false.
`TestJudgementGate` (tier0) covers the entry passing a valid file and failing on a missing file and on a file carrying another spawn id.
`TestSeatTables` gains the judge table's rows, autonomous and interactive; the resolver tests and the stencil marker test gain the new names.

**ImpactSummary:** Adds the judge seat, gate name, stencil, snapshot and three Deps fields wired in loomcli; one new lymdirs constant; nothing runs it until Drift-Verify does.
