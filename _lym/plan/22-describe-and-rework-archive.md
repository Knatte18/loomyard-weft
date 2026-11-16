# Card 22 — describe-and-rework-archive

**Create:**
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

**Uses:**
- `plan:internal/weaverengine#ArchiveRunRecord`
- `plan:internal/weaverengine#Active`
- `plan:internal/weaverengine#RunDirs`
- `plan:internal/weaverengine#DirRel`
- `internal/websterengine#ArchiveRunRecord`
- `internal/loomcli/doc.go`

**Intent:** Describe reads the active implementer's run record and every archived one, and a PR rework round archives Weaver's run directory into its own slot (D9, D10, D13).

- `ArchivedRunRecordDirs` lists each round's archived implementer directories under `reworkDir`, ascending by round and, within a round, `webster` before `weaver`; a round holding neither is skipped and any other read or stat failure is an error, as `ArchivedWebsterDirs` did.
  The prior-generation slot names sit beside `reworkPriorWebster` as one const block, `weaver` added.
- `archiveRunRecords` is the rework archive seam's closure (`ArchiveWebster`): it archives Webster's run directory into the slot it is handed and Weaver's into the `weaver` slot beside it (`weaverengine.ArchiveRunRecord`), so the next generation of either starts fresh.
- `activeRunRecordPath` returns `summary.md` under the active implementer's run directory (`weaverengine.Active` and `RunDirs`); a seam refusal is returned, never read as Webster's.
- `loomCLI.wire` sets `ArchiveWebster` to `archiveRunRecords` and has `DescribeTable` take `RunRecordPath` from `activeRunRecordPath` and `PriorRunRecordPaths` from `ArchivedRunRecordDirs`.
- `planCommitPathspecWith` stages Weaver's run directory by the rule it already applies to Webster's: when it holds a file or git tracks a path under it, so the plan commit after a rotation stages Weaver's archive deletions too.
- `TestInterruptPolicies_ClosedVocabulary`'s keep reason names `TestArchivedRunRecordDirs` in place of the deleted test.
- `loomshed/doc.go`'s rework-layout paragraph names both slots.

`TestArchivedRunRecordDirs` is tier0: rounds 1, 2 and 10 list in numeric order with a round's Webster slot before its Weaver slot; a round with neither is skipped; an absent dir lists none; a file as the rework dir errors.
`TestImplementerRunRecords` is tier0 over temp anchors: `archiveRunRecords` moves a planted `_lym/weaver/` into the destination's `weaver` slot and `_lym/webster/` into the destination; `activeRunRecordPath` names Weaver's `summary.md` with Weaver begun, Webster's with Webster begun, and returns the seam's refusal with both begun.
`TestPlanCommitPathspec` gains rows: a Weaver directory holding a file, and a tracked one emptied by an archive, each add `_lym/weaver`.

**ImpactSummary:** Replaces one loomshed lister whose only caller is loom's Describe wiring, moves two wiring closures into tested helpers, widens the plan commit's pathspec to Weaver's directory and repoints one keep reason.
