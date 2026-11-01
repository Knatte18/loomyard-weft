# Card 16 — parts-producers

**Create:**
- `plan:internal/shedrecipe#PartSeats` -> `type PartSeats struct`
- `plan:internal/shedrecipe#partsWriteEntry` -> `func partsWriteEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#partsPlanWriteEntry` -> `func partsPlanWriteEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#resolvePartGates` -> `func resolvePartGates(entry string, cfg Config, env Env) (func(part string) (shuttleengine.GateSpec, error), error)`
- `plan:internal/shedrecipe#TestPartsWriteEntry` -> `func TestPartsWriteEntry(t *testing.T)`
- `plan:internal/shedrecipe#TestPartsPlanWriteEntry` -> `func TestPartsPlanWriteEntry(t *testing.T)`
- `plan:internal/loomcli#partsPlanCommitPathspec` -> `func partsPlanCommitPathspec(location *lymcwd.Location) []string`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/shedrecipe#TestResolveGateSpec_UnknownNameListsTheLegalValues`
- `internal/shedrecipe#TestResolveGateSpec_Rejections`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#gateCapableEntries`
- `internal/loomcli#TestPlanCommitPathspec`
- `internal/testkit/envkit#FullEnv`
- `internal/battencli#intentionallyNil`
- `internal/loomcli#loomCLI.wire`

**Extends:**
- `internal/shedrecipe/doc.go`

**Uses:**
- `plan:internal/loomengine#PartsTable`
- `plan:internal/loomengine#PartsPlanTable`
- `plan:internal/loomshed#NewPartsGate`
- `plan:internal/loomshed#NewPartPlanGate`
- `plan:internal/loomshed#NewPartAccept`
- `plan:internal/loomshed#NewPartsDirRotator`
- `plan:internal/loomshed#NewPartsPlanRotator`
- `internal/partsparser#DirRel`
- `internal/loomshed#NewPlanWrite`
- `internal/shedrecipe#planContextOf`
- `internal/shedrecipe#resolveGateSpec`
- `internal/loomcli#planCommitPathspec`
- `internal/shedrecipe#TestGateCapableEntries_RejectLeftoverOldKeys`
- `internal/shedrecipe#TestPlanWriteEntry_ConstructionFailures`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`

**Intent:** Register the two producers (D4, D6, D13) and wire their seams, so a recipe row with engine `PartsWrite` or `PartsPlanWrite` builds.

**Env.** `Env` gains, each with a doc comment naming its reader:
- `PartsTable shedadapters.TableSource`, the `Parts-Write` lead's table, evaluated per call;
- `PartsPlanTable func(PartSeats) (seatengine.Table, error)`, the `Parts-Plan-Write` table, told the per-part hooks only the entry can build;
- `CommitParts func() error`, committing `_lym/parts/`;
- `CommitPartsPlan func() error`, committing the plan's pathspec and `_lym/parts/`.
`PartSeats` holds `Gates func(part string) (shuttleengine.GateSpec, error)`, `Accept func(part string) func(shuttleengine.Result) error` and `FindingsPath func(part string) string`.

**Gate vocabulary.** `resolvePlainGateClosure` gains `parts`: it requires `AnchorPath` and `DecisionRecordPath` absolute and a non-nil `Setting` seam, reads the cap with `env.Setting("tapestry", "parts_max")` as a row value is looked up by key (PATTERN-shed-recipe-registry), refuses a value that is not an integer of 1 or more with an error naming `tapestry.parts_max`, and returns `loomshed.NewPartsGate(env.AnchorPath, env.DecisionRecordPath, max)`.
Its unknown-name error lists `parts` among the legal names; `TestResolveGateSpec_UnknownNameListsTheLegalValues` asserts the new list, `TestResolveGateSpec_ValidOneEntryPerVocabularyName` gains a `parts` row whose env sets a `Setting` answering 6 for `tapestry.parts_max`, and `TestResolveGateSpec_Rejections` gains a row per `parts` requirement: an empty `AnchorPath`, an empty `DecisionRecordPath`, a nil `Setting` and a `Setting` answering 0 for `tapestry.parts_max`, each refused naming its source.
`resolvePartGates` reads the row's `part_gates` key as `resolveGateSpec` reads `gates`, with the same element keys, but its one legal name is `plan-part`, which requires `AnchorPath` and `WorktreeRoot` absolute and a non-nil `PlanIndex`; any other name is refused naming `plan-part`.
It validates the list once at construction and returns a closure that builds, for a part, one `GateEntry` per element whose gate is `loomshed.NewPartPlanGate(env.AnchorPath, env.WorktreeRoot, env.RunScratchDir, part, env.PlanIndex, planContextOf(env))`.

**Entries.** `registry` gains `"PartsWrite": partsWriteEntry` and `"PartsPlanWrite": partsPlanWriteEntry`.
- `partsWriteEntry` accepts only the `gates` key, requires `PartsTable`, `CommitParts`, `Seats` and an absolute `AnchorPath`, and returns `loomshed.NewPlanWrite(row.Name, inner, env.CommitParts)` over a sourced `MultiLLMProducer` whose table source stamps the resolved gate onto `env.PartsTable()`'s table, whose preparation is `loomshed.NewPartsDirRotator(env.AnchorPath, env.Now)`, and which keeps gate findings at `env.gateFindingsPath(row.Name)`.
- `partsPlanWriteEntry` accepts the `gates` and `part_gates` keys, requires `PartsPlanTable`, `CommitPartsPlan`, `Seats`, `PlanIndex`, `Rework.ArchiveWebster` and absolute `AnchorPath`, `WorktreeRoot` and `StencilsDir`, and returns `loomshed.NewPlanWrite(row.Name, inner, env.CommitPartsPlan)` over a sourced producer whose table source calls `env.PartsPlanTable` with `PartSeats{Gates: <resolvePartGates' closure>, Accept: loomshed.NewPartAccept over the same roots, index and plan context, FindingsPath: env.gateFindingsPath(row.Name + "-" + part)}` and stamps the resolved `gates` onto its chair, whose preparation is `loomshed.NewPartsPlanRotator(env.AnchorPath, env.StencilsDir, env.Rework.ArchiveWebster, env.Now)`, and which keeps the chair's gate findings at `env.gateFindingsPath(row.Name)`.
Both reuse `loomshed.NewPlanWrite`'s decorator, which commits on Done and on a gate-failed or writer Stuck alike whenever the pointer is non-empty.

**Wiring.** `loomCLI.wire` fills the four seams for every recipe: `PartsTable` calls `loomengine.PartsTable` over the location, the stencils and specs directories, the loaded loom config and the model registry; `PartsPlanTable` calls `loomengine.PartsPlanTable` over the same, the gate tiers and shuttle's loaded `run_timeout_min` as a duration, passing the told `PartSeats` fields; `CommitParts` commits `partsparser.DirRel()` with the message `loom: parts artifacts for <slug>`; `CommitPartsPlan` commits `partsPlanCommitPathspec(location)`, which is `planCommitPathspec(location)` followed by `partsparser.DirRel()`, with the plan commit's message `loom: plan artifacts for <slug>`.
`TestPlanCommitPathspec` gains rows for `partsPlanCommitPathspec`: the plan directory then the parts directory, and webster's directory between them when it holds a file a rotation moved there.
`envkit.FullEnv` fills the four seams with a one-lead table, a one-chair table and two no-op commits, so `NilSeams(FullEnv)` stays empty; batten wires no tapestry row, so `intentionallyNil` lists the four seams as loom-only.
The doc gains both entries and the two gate names.
`TestNames` gains `PartsPlanWrite` and `PartsWrite` in its sorted list, `minimalEntries` gains a minimal Env and Config for each new entry so the conformance test builds it, and `gateCapableEntries` gains both entries so `TestGateCapableEntries_RejectLeftoverOldKeys` checks their leftover-key rejection.

`TestPartsWriteEntry` (tier0, over `shedfake.SeatRunner`) covers each missing seam refused naming its field, an unknown key refused, a Done committing once, a gate-failed Stuck committing with the findings kept, and the fresh path running the preparation before the run.
`TestPartsPlanWriteEntry` (tier0) covers the same refusals, a `part_gates` element of another name refused naming `plan-part`, the table source handed hooks that give each part a `plan-part` gate, a findings path `<row>-<part>` and an Accept, the chair stamped with the `plan` gate, and a Done and a writer Stuck each committing once.

**ImpactSummary:** Two registry rows, four Env seams filled by loomcli and envkit, and one gate name; existing entries and rows are unchanged.
