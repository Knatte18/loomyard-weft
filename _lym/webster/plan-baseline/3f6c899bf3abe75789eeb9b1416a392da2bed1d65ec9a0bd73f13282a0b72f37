# Card 13 — drift-expect-rows

**Create:**
- `internal/loomshed#NameDriftExpect`
- `internal/loomrecipe#driftExpectType`

**Edit:**
- `contracts/recipes/darn-recipe.yaml`
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/loomshed#InterruptPolicies`
- `internal/loomshed/doc.go`
- `internal/reportengine#loomPhases`
- `internal/reportengine#darnPhases`
- `internal/reportengine#darnRowPhases`
- `internal/reportengine#TestComputeTiming`
- `internal/reportengine/doc.go`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#wantDarnProducerTable`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#deriveSequenceOrder`
- `internal/loomrecipe#TestDriftRows_EntryGate`
- `internal/loomrecipe#TestEntryNumbers`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`
- `internal/loomrecipe#gatedRowFanViolations`
- `internal/loomcli#TestRenderChecklist`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml`
- `internal/loomrecipe/doc.go`

**Uses:**
- `internal/loomshed#TestInterruptPolicies_ClosedVocabulary`

**Intent:** Put Drift-Expect into both recipes (D4): in tapestry after Drift-Await and before Webster, in darn after Drift-Await and before Darn-Write, so its agent writes against the counted capture's real cases and identities.

The row: `name: Drift-Expect`, `engine: DriftExpect`, `entry_gate: drift.enabled`, no `on_stuck`, `config: {subject: decision-record, gates: [{name: expectation, attempts: 3}]}`, Drift-Await's `on_done` naming it and its own `on_done` naming the writer.
The `loomrecipe` package doc's row orders of both recipes gain Drift-Expect after Drift-Await.
Both recipe headers name the row in the main line, list it among the rows whose Stuck halts the run, and say a run parked before it reaches it while one already past it has no seal and is checked against an empty expectation (D4, D6).

`NameDriftExpect` joins the drift row names, and `InterruptPolicies` gives it `InterruptPolicyReinvoke` like the other drift rows.
The timing report counts it in the setup phase in both recipes, as `darnRowPhases` and `TestComputeTiming` pin.
`gatedRowFanViolations` admits `DriftExpect` as a gated writer engine, which sets no fork subagents.
The recipe tests move to the new row order and edges: the producer tables, whose Drift-Expect rows read `driftExpectType` (`reflect.TypeOf(&driftengine.DriftExpect{})`, declared in `shape_test.go`'s type block), darn routing, the progress and sequence orders, the entry-gate rows, entry numbers, row names, and the checklist's numbering; `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` gains `DriftExpect`, and `TestTapestryMotifBlock`'s fixture `tapestry-rows-before-motif.yaml` gains the row.

**ImpactSummary:** Inserts one row into each shipped recipe; row names and positions pinned by the recipe, timing and checklist tests move by one, and in-flight runs keep resume by row name.
