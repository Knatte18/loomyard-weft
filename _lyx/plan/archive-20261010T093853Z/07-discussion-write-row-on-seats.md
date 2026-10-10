# Card 7 — discussion-write-row-on-seats

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#TestDiscussionWriteEntry_ConstructionFailures`
- `internal/shedrecipe#TestDiscussionWriteEntry_CallDone`
- `internal/shedrecipe#TestDiscussionWriteEntry_ParentReviewBeginRound`
- `internal/shedrecipe/doc.go`
- `internal/testkit/envkit#FullEnv`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/loomrecipe#newLoomShuttle` -> `func newLoomShuttle(planDir string) *shedfake.Shuttle`
- `internal/loomrecipe#runLoomShuttle` -> `func runLoomShuttle(planDir string, spec shuttleengine.Spec) (shuttleengine.Result, error)`
- `internal/loomrecipe/resume_test.go`
- `internal/loomrecipe/shape_test.go`
- `internal/loomrecipe/sequence_test.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/parentreview_integration_test.go`
- `internal/battencli#intentionallyNil`
- `internal/shedbuild#engineMinimalConfig`
- `internal/loomshed#discussionWrite`
- `internal/loomshed#discussionWrite.Call`

**Delete:**
- `internal/loomcli#TestWire_DiscussionSpecEvaluatesToExpectedShape`

**Create:**
- `plan:internal/loomrecipe#newLoomSeats` -> `func newLoomSeats(writeOutputs bool) *shedfake.SeatRunner`
- `plan:internal/loomcli#TestWire_DiscussionTableEvaluatesToExpectedShape` -> `func TestWire_DiscussionTableEvaluatesToExpectedShape(t *testing.T)`

**Uses:**
- `plan:internal/loomengine#DiscussionTable`
- `plan:internal/shedadapters#NewMultiLLMProducerSourced`
- `plan:internal/shedadapters#TableSource`
- `internal/parentreview#Store.PrepareRound`
- `internal/loomshed#NewDiscussionWrite`
- `internal/testkit/shedfake#SeatRunner`

**Intent:** Run the Discussion-Write row on `MultiLLMProducer` (D1, D2, D3, D9), keeping the row name, the engine name and the commit decorator.
`Env.DiscussionSpec` is replaced by `Env.DiscussionTable func() (seatengine.Table, error)`, a closure the entry evaluates once per `Call`; its doc keeps the reason it is a closure.
`discussionWriteEntry` requires `DiscussionTable`, `CommitDiscussion` and `Seats`, no longer `Shuttle`, resolves the row's gates as today, wraps the source so the evaluated table carries the resolved gate, wires `PrepareRound` as a `func() error` preparation exactly when the gate list holds an enabled `parent-review` entry and leaves it nil otherwise, and returns `loomshed.NewDiscussionWrite` over `NewMultiLLMProducerSourced`; its doc comment states the new shape and that zero advisors is the single-agent case.
Bound: a nil preparation skips nothing the row ran before, since without an enabled `parent-review` gate the single-agent row wired none either.
`newTestEnv` and `FullEnv` fill `DiscussionTable` with a one-chair table over one absolute output path under their temp root and the chair stencil name.
`TestDiscussionWriteEntry_ConstructionFailures` checks the nil `DiscussionTable` and nil `Seats` seams; `TestDiscussionWriteEntry_CallDone` drives the fake seat runner and asserts the source was evaluated once, the table's gate is the row's resolved gate, the pointer is the chair's first output and the commit fired once; `TestDiscussionWriteEntry_ParentReviewBeginRound` keeps its four cases over the seat fake, the attach case scripting a live chair and a resume result, and gains a case where the preparation fails and no seat runs and nothing is archived.
In `loomrecipe`, `buildSequenceFixture` fills `DiscussionTable` with a chair over the fixture's two discussion paths and `Seats` with `newLoomSeats(true)`, whose `RunFn` writes the chair's outputs when told to and reports a done chair with no `GateOutcome`, so card 6's fake evaluates Discussion-Write's own gate; `newLoomShuttle` and `runLoomShuttle` lose the discussion branch and the `writeOutputs` flag, `shape_test.go` takes `newLoomSeats(false)` for its blocked run, and the two Discussion-Write resume tests assert the seat fake's probe, run and resume counts instead of the shuttle's attach and discussion-role run counts, the live-chair case scripting `ProbeFn` with a non-nil chair handle and `ResumeFn` with a done result; the comments in `shape_test.go` and `sequence_test.go` that name the fake shuttle for row 3 name the seat fake.
In `loomcli`, `wire` replaces the `DiscussionSpec` closure with a `DiscussionTable` closure over `DiscussionTable(location, websterGeom.StencilsDir, loomCfg, registry, seedSlug(...))`, keeping the comment on per-call evaluation; `TestWire_DiscussionTableEvaluatesToExpectedShape` evaluates it for both `discussion_interactive` values and asserts the table's interactivity, role prefix, segment, timeout, the chair's skills, outputs and non-empty model, one advisor per template entry, and, through `seatengine.Prompts` over the wired geometry, a chair prompt naming the parent with no unrendered marker; the old test is deleted.
`parentreview_integration_test.go` replaces `scriptedShuttle` with a `shedfake.SeatRunner` whose `RunFn` runs the test's script over `table.Gate` and returns the chair's result carrying the `GateOutcome` the script built, so the fake never evaluates the gate a second time; it fills `Env.Seats` and `Env.DiscussionTable` on the fixture and keeps every assertion of the exchange.
`intentionallyNil` renames its `DiscussionSpec` key to `DiscussionTable`.
The comment of `engineMinimalConfig` says DiscussionWrite wraps the multi-LLM producer and names `DiscussionTable` as its injected closure; the doc comments of `discussionWrite` and `discussionWrite.Call` name the wrapped producer as a `*MultiLLMProducer` in practice, whose `Call` entry-checks the context first and reports a non-empty pointer only for Done or a gate-failed Stuck.
The shedrecipe package doc gains a paragraph on the DiscussionWrite entry's told table and preparation.

**ImpactSummary:** One Env field replaced, touching every filler of it across shedrecipe, envkit, loomrecipe, loomcli and battencli, plus comment-only edits in shedbuild and loomshed; the recipe row's name, engine and gates are unchanged, and the single-agent symbols survive unreferenced until card 8.
