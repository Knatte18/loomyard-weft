# Card 8 — The darn writer stops on a condition its task names

**Create:**
- `internal/shedrun#ConditionStopMarkerFileName`
- `internal/shedrun#ConditionStopMarker`

**Edit:**
- `internal/loomshed#DarnDeps`
- `internal/loomshed#darnWrite.Call`
- `internal/loomshed#darnWrite.checkSeams`
- `internal/loomshed#newDarnFixture`
- `internal/loomshed#darnFixture`
- `internal/loomshed#darnSession.Call`
- `internal/loomshed#TestDarnWrite_Call`
- `internal/loomshed/doc.go`
- `internal/darnengine#DarnInputs`
- `internal/darnengine#DarnTable`
- `internal/darnengine#newSpecInputs`
- `internal/darnengine#TestDarnSpec`
- `internal/darnengine/doc.go`
- `internal/loomcli#darnDeps`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/shedrun#TestPathConstructors`
- `internal/testkit/envkit#FullEnv`
- `internal/shedrecipe#darnWriteEntry`
- `internal/shedrecipe#darnTestEnv`
- `internal/shedrecipe#TestDarnWriteEntry_ConstructionFailures`
- `internal/loomcli#TestWire_DarnRecipe`
- `contracts/stencils/darn/darn-template-write.md`

**Uses:**
- `internal/shedrun#StopMarker`
- `internal/shedrun/doc.go`
- `internal/loomcli/doc.go`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/shedrecipe#TestDarnWriteEntry_HappyPath`
- `internal/shedrecipe/doc.go`

**Intent:**
D9: the darn writer gains a second stop, on a condition the task's body names, told apart from the outdated-entry stop by a second marker.

`shedrun.ConditionStopMarker` returns `ConditionStopMarkerFileName` under `RunDir(l, runID)`, beside `StopMarker`, with the same doc shape.

Stencil `darn-template-write.md`: Step 3 gains a second stop after the outdated-entry one: when the task's body names a stop condition and the writer finds it holds, it writes its one output file as the stop report with the content the body asks for, in the final-summary spec's form, creates the stop marker at `{{.stop_marker_path}}` and the second marker, an empty file, at `{{.condition_stop_marker_path}}`, and ends its turn with no commit.
It never stops this way on a condition it infers: only a condition the body's own text names.
The header comment names `condition_stop_marker_path` as a required marker.
PATTERN-stencil-ownership: the stencil states the stop; the body only names its condition and what the report holds.

`darnengine.DarnInputs` gains `ConditionStopMarkerPath`, required like `StopMarkerPath`, and `DarnTable` puts it in `Values` as `condition_stop_marker_path`; `loomCLI.wireDarn` fills it with `shedrun.ConditionStopMarker(location, c.runID)`.

`loomshed.DarnDeps` gains `ConditionStopMarkerPath`, which `checkSeams` names when unwired and `loomcli.darnDeps` fills with `shedrun.ConditionStopMarker(l, runID)`; `envkit.FullEnv` fills it under the fixture's dir, so the seam tests listed under Uses read it filled.
`shedrecipe.darnWriteEntry` adds `Darn.ConditionStopMarkerPath` to the paths it requires as absolute and rooted, beside `Darn.StopMarkerPath`, and `darnTestEnv` fills it under its dir, so `TestDarnWriteEntry_HappyPath`, which runs the producer's `Call`, passes `checkSeams`.
`darnWrite.Call` removes the condition marker before each session, as it removes the stop marker.
On a stop marker the session wrote, it reads the condition marker: present, the halt reason is the stop report alone, whose own way forward stands, and `darnStopWayForward` is not appended; absent, the reason is the report and `darnStopWayForward` as today.
A condition marker with no stop marker is no stop.
The run then halts Stuck and the operator is told, as for any darn stop.

Tests:
- `TestDarnWrite_Call` gains rows, its fixture's session writing the condition marker when asked: a stop marker with the condition marker halts with the report alone, not naming `lym loom resume`; a stale condition marker is removed before the session; a condition marker alone does not halt; an unwired `ConditionStopMarkerPath` is named.
- `TestDarnSpec` asserts the table carries `condition_stop_marker_path` and that an empty `ConditionStopMarkerPath` is refused.
- `TestPathConstructors` gains the row of `ConditionStopMarker`.
- `TestDarnWriteEntry_ConstructionFailures` gains the row of a relative `ConditionStopMarkerPath`, refused naming the field.
- `TestWire_DarnRecipe`'s Darn path table gains `ConditionStopMarkerPath`, wanting `shedrun.ConditionStopMarker(loc, c.runID)`.

`loomshed/doc.go` and `darnengine/doc.go` state the second marker and its halt reason.

**ImpactSummary:** one new marker path threaded from shedrun through darnengine, loomcli, shedrecipe's entry check and the Darn-Write producer, plus the stencil's second stop; the outdated-entry stop keeps its reason.

**Commit:** 8: darn writer stops on a condition the task body names
