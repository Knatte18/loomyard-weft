# Card 22 — describe-and-rework-archive

**Create:**
- `plan:internal/loomshed#ReworkPriorWeaver` -> `const ReworkPriorWeaver = "weaver"`
- `plan:internal/loomshed#ArchivedRunRecordDirs` -> `func ArchivedRunRecordDirs(reworkDir string) ([]string, error)`
- `plan:internal/loomshed#TestArchivedRunRecordDirs` -> `func TestArchivedRunRecordDirs(t *testing.T)`
- `plan:internal/loomcli#archiveRunRecords` -> `func archiveRunRecords(webster websterengine.Geometry, weaver weaverengine.Geometry) func(dest string) error`
- `plan:internal/loomcli#activeRunRecordPath` -> `func activeRunRecordPath(active weaverengine.ActiveDeps) (string, error)`
- `plan:internal/loomcli#TestImplementerRunRecords` -> `func TestImplementerRunRecords(t *testing.T)`

**Delete:**
- `internal/loomshed#ArchivedWebsterDirs`
- `internal/loomshed#TestArchivedWebsterDirs`
- `internal/loomshed#TestArchivedWebsterDirs_UnreadableDirErrors`

**Edit:**
- `internal/loomshed#reworkPriorWebster`
- `internal/loomshed#TestInterruptPolicies_ClosedVocabulary`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#planCommitPathspecWith`
- `internal/loomcli#TestPlanCommitPathspec`
- `internal/loomshed/doc.go`
- `internal/failuretrace#NewTracer`
- `internal/failuretrace#Input`
- `internal/failuretrace#Trace`
- `internal/failuretrace#lastCardSHA`
- `internal/failuretrace/doc.go`
- `internal/failuretrace#TestTrace`
- `internal/failuretrace#TestNewTracer`

**Uses:**
- `plan:internal/weaverengine#ArchiveRunRecord`
- `plan:internal/weaverengine#Active`
- `plan:internal/weaverengine#RunDirs`
- `plan:internal/weaverengine#DirRel`
- `internal/websterengine#ArchiveRunRecord`
- `internal/loomcli/doc.go`

**Intent:** Describe and the landing failure tracer read the active implementer's run record, Describe every archived one too, and a PR rework round archives Weaver's run directory into its own slot (D9, D10, D13).

- `ArchivedRunRecordDirs` lists each round's archived implementer directories under `reworkDir`, ascending by round and, within a round, `webster` before `weaver`; a round holding neither is skipped and any other read or stat failure is an error, as `ArchivedWebsterDirs` did.
  `ReworkPriorWeaver` sits beside `reworkPriorWebster` in the slot const block, exported because loomcli names the same slot, and `ArchivedRunRecordDirs` reads it.
- `archiveRunRecords` is the rework archive seam's closure (`ArchiveWebster`): it archives Webster's run directory into the slot it is handed and Weaver's into the slot `loomshed.ReworkPriorWeaver` names, joined onto `dest`'s parent directory (`weaverengine.ArchiveRunRecord`), so the next generation of either starts fresh.
- `failuretrace.NewTracer`, which loom wires into Parent-Align and Finalize, reads the active implementer's `runplan.State` through `weaverengine.Active` over the anchor with a nil `Setting`, and returns a seam refusal as its error, so a Weaver run's verify-failure records keep their card attribution.
  `Input`'s state field, `Trace` and `lastCardSHA` take `*runplan.State` and `*runplan.BatchState`; the nil-state error and `failuretrace/doc.go` name the implementer's state, not Webster's.
- `activeRunRecordPath` returns `summary.md` under the active implementer's run directory (`weaverengine.Active` and `RunDirs`); a seam refusal is returned, never read as Webster's.
- `loomCLI.wire` sets `ArchiveWebster` to `archiveRunRecords` and has `DescribeTable` take `RunRecordPath` from `activeRunRecordPath` and `PriorRunRecordPaths` from `ArchivedRunRecordDirs`.
- `planCommitPathspecWith` stages Weaver's run directory by the rule it already applies to Webster's: when it holds a file or git tracks a path under it, so the plan commit after a rotation stages Weaver's archive deletions too.
- `TestInterruptPolicies_ClosedVocabulary`'s keep reason names `TestArchivedRunRecordDirs` in place of the deleted test.
- `loomshed/doc.go`'s rework-layout paragraph names both slots.

`TestArchivedRunRecordDirs` is tier0: rounds 1, 2 and 10 list in numeric order with a round's Webster slot before its Weaver slot; a round with neither is skipped; an absent dir lists none; a file as the rework dir errors.
`TestImplementerRunRecords` is tier0 over temp anchors: `archiveRunRecords` moves a planted `_lym/weaver/` into the destination's `weaver` slot and `_lym/webster/` into the destination; `activeRunRecordPath` names Weaver's `summary.md` with Weaver begun, Webster's with Webster begun, and returns the seam's refusal with both begun.
`TestPlanCommitPathspec` gains rows: a Weaver directory holding a file, and a tracked one emptied by an archive, each add `_lym/weaver`.
`TestNewTracer` gains a row that traces a failure against a planted Weaver state; `TestTrace`'s nil-state row reads the new error text.

**ImpactSummary:** Replaces one loomshed lister whose only caller is loom's Describe wiring, moves two wiring closures into tested helpers, widens the plan commit's pathspec to Weaver's directory, repoints one keep reason, and moves the failure tracer's state read and input type onto the seam and `runplan.State`.
