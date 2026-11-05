# Card 12 — drift-expect-producer

**Create:**
- `plan:internal/driftengine#ExpectInputs` -> `type ExpectInputs struct`
- `plan:internal/driftengine#ExpectTable` -> `func ExpectTable(in ExpectInputs, cfg Config, reg modelspec.Registry) (seatengine.Table, error)`
- `plan:internal/driftengine#NewExpectationGate` -> `func NewExpectationGate(d Deps) shuttleengine.Gate`
- `plan:internal/driftengine#DriftExpect` -> `type DriftExpect struct`
- `plan:internal/driftengine#NewDriftExpect` -> `func NewDriftExpect(name string, row Row, expectation shuttleengine.GateEntry, deps Deps) *DriftExpect`
- `plan:internal/driftengine#DriftExpect.Call` -> `func (p *DriftExpect) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/driftengine#TestDriftExpect` -> `func TestDriftExpect(t *testing.T)`
- `plan:internal/driftengine#expectFixture` -> `type expectFixture struct`
- `plan:internal/driftengine#newExpectFixture` -> `func newExpectFixture(t *testing.T) *expectFixture`
- `plan:internal/shedrecipe#driftExpectEntry` -> `func driftExpectEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:contracts/stencils#DriftTemplateExpect` -> `var DriftTemplateExpect string`
- `contracts/stencils/drift/drift-template-expect.md`

**Extends:**
- `internal/driftengine#Deps`
- `internal/driftengine#Paths`
- `internal/driftengine/doc.go`
- `internal/shedrecipe/doc.go`

**Edit:**
- `internal/driftengine#NewPaths`
- `internal/driftengine#TestSeatTables`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#TestDriftEntries`
- `internal/shedrecipe#driftTestEnv`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/shedrecipe#TestResolveGateSpec_UnknownNameListsTheLegalValues`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `internal/loomcli#driftDeps`
- `internal/loomcli#driftTables`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#driftDeps`
- `internal/battencli#intentionallyNil`
- `internal/shedbuild#engineMinimalConfig`

**Uses:**
- `plan:internal/driftengine#commitStore`
- `plan:internal/driftengine#commitStore.deps`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `plan:internal/driftengine#ListCaptures`
- `plan:internal/driftengine#ParseExpectation`
- `plan:internal/driftengine#SealExpectation`
- `plan:internal/driftengine#CheckBaseline`
- `plan:internal/driftengine#WriteRecords`
- `plan:internal/driftengine#ReadRecords`
- `plan:internal/driftengine#DeferHaltPrefix`
- `internal/driftengine#gateLoopSeats`

**Intent:** The Drift-Expect producer (D4, D5, D6) and its `expectation` gate entry, registered but in no recipe yet (card 13).

- `Paths` gains `Listing`, `.lym/drift/listing.yaml`, the identity listing the agent reads, declared in `NewPaths`.
- `Deps` gains `ExpectTable func(listing string) (seatengine.Table, error)` and `CommitsSince func(base string) ([]string, error)`, the warp commits after `base`; `loomcli`'s `driftDeps` wires them over `ExpectTable` and `Fabric.CommitLog`, `driftTables` gains the plan directory, `envkit`'s `driftDeps` fills fakes, and `battencli`'s `intentionallyNil` lists both.
- `ExpectTable` builds one lead seat on the new `drift-template-expect` stencil, role `drift-expect`, the plan segment, never interactive, model `cfg.ExpectModel`, skill `scribe:prose`, its one output `expected.yaml`, with values for the task's subject reading (decision record, darn's record being the same file), the plan directory under tapestry, the listing path, the expectation path and the diff mode; skills and colour ride the launch spec (`PATTERN-parent-directive`).
- `NewExpectationGate` parses `expected.yaml` with `ParseExpectation` under the recorded mode, or config's when the record holds none, against `ListCaptures` of the baseline, failing with the parser's message, which re-prompts the agent; a missing file fails asking for it.
- `DriftExpect.Call`, after `passWhenSwitchedOff`, is Done at once with no scope record, scope `off`, a deferred capture, and a seal commit (D4, D6, D12); a Done without sealing on a run with a scope record records `Unsealed` through `WriteRecords`.
  A record with no `Diff`, from a run seeded before this lands, records config's `diff` and `key_patterns` through `WriteRecords` first and keeps them (D3).
  Otherwise:
  - a record with `awaited_at` set and not deferred whose baseline directory is missing, read through `CheckBaseline` and never through `capture.json` (Shared Decisions), is a Stuck whose reason opens with `shedadapters.DecisionHaltPrefix` then `DeferHaltPrefix`, naming `lym drift defer` (D12); a record with no `awaited_at` is a Stuck the same way, saying the capture never counted;
  - a warp HEAD other than the capture's head, the scope record's recorded `BaselineHead` falling back to `capture.json`'s head, is a Stuck whose reason opens with `DecisionHaltPrefix` then `DeferHaltPrefix`, naming the commits `CommitsSince` lists and two ways forward: the parent's `lym drift defer`, or the operator resetting the run's warp branch to the baseline head in the run's worktree followed by `lym loom resume` (D6);
  - an `expect_session` mark still open probes the agent's table, resumes a live chair and respawns otherwise; a closed mark with an `expected.yaml` that validates seals it with no spawn, and one with a file that does not validate spawns the agent, whose entry re-prompts on the findings;
  - otherwise it writes the listing (every baseline case's identities and text under `lines`, the case names alone under `command`, D2), opens the mark through `WriteRecords`, and runs the agent through `shedadapters.NewMultiLLMProducerSourced` with the table gated by the row's `expectation` entry; the session's return closes the mark, a Done seals the file through `SealExpectation`, carrying the `Drift-Baseline` digest trailer when `CheckBaseline` reports `Unguarded` (D7), and an exhausted entry is the shed's Stuck, which the stop chain handles and whose resume spawns the agent again (D5).
  Every Stuck that names a parent verb or the warp-branch reset opens with `DecisionHaltPrefix`, so the stop chain hands it to the parent and never to the arbiter (`PATTERN-shed-stop-chain`).
- `driftExpectEntry` reads the required `subject` and a `gates` list of exactly one entry named `expectation`, refuses any other key with the Drift key way forward, checks `Env.Seats` and the seams it reads, and builds the producer; `registry` maps `DriftExpect` to it, and `resolvePlainGateClosure` resolves `expectation` to `NewExpectationGate` over `driftDepsFrom(env)` (`PATTERN-shed-recipe-registry`).
- The stencil reads the listing, the capture files and the task's record and plan, and writes `expected.yaml` per D5, or per D2 under `command`: what it expects to change and why, never what may be accepted, with no tolerance; a stated output invariant yields an empty expectation.
  It carries the parent and edit directive markers, and `roleOpeningStencils` maps `drift-expect` to it.

`TestDriftExpect` (tier0, `t.Parallel`) runs on `expectFixture`, its own fixture over `commitStore`, a `t.TempDir` baseline and `gateLoopSeats` for the agent, and covers: Done with no agent under scope `off`, a deferral and an existing seal commit; one weft commit carrying `Drift-Seal` after a Done agent; with no `Drift-Baseline` commit yet, that seal commit also carries `Drift-Baseline` equal to the baseline's digest; a valid unsealed file with a closed mark seals without a spawn, and an invalid one spawns the agent; an open mark probes before sealing and resumes a live chair; a moved warp head is a Stuck naming the commits and `lym drift defer`; a missing baseline directory with `awaited_at` set, no `capture.json` and no `Drift-Baseline` commit is a Stuck naming `lym drift defer`; a scope record with no `awaited_at` is a Stuck saying the capture never counted and naming `lym drift defer`, spawning no agent and writing no seal commit; each of those three Stucks' reasons opens with `DecisionHaltPrefix` followed by `DeferHaltPrefix`; the agent is handed every baseline case's listing, and under `command` the case list only; a Done without sealing records `Unsealed`; a record with no `Diff` records config's mode and patterns; an `expectation` entry that fails every attempt ends the call Stuck, and the next call spawns the agent again.
A `TestDriftExpect` sub-run drives `NewExpectationGate` alone: a valid file passes, a missing file fails asking for it, and an unknown case fails with the parser's message.
`TestSeatTables` gains the expect table's row; `TestDriftEntries`, `TestNames`, `minimalEntries` and the resolver tests gain `DriftExpect` and `expectation`; `engineMinimalConfig` gains `DriftExpect`'s `subject` and one `expectation` gate, since the entry builds with no less; `TestStencils_TopLevelMarkers` gains the stencil's markers.

**ImpactSummary:** Adds a registered engine, a gate name, a stencil and two Deps seams wired in loomcli and envkit; no recipe runs the row yet.
