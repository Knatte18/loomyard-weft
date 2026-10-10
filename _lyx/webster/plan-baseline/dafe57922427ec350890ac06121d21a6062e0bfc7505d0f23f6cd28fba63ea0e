# Card 7 — discussion-seats-entry

**Create:**
- `plan:internal/shedrecipe#discussionSeatsEntry` -> `func discussionSeatsEntry(name string, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#TestDiscussionSeatsEntry_ConstructionFailures` -> `func TestDiscussionSeatsEntry_ConstructionFailures(t *testing.T)`
- `plan:internal/shedrecipe#TestDiscussionSeatsEntry_Call` -> `func TestDiscussionSeatsEntry_Call(t *testing.T)`
- `plan:internal/loomcli#TestWire_DiscussionTableEvaluatesToExpectedShape` -> `func TestWire_DiscussionTableEvaluatesToExpectedShape(t *testing.T)`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`
- `internal/shedrecipe/doc.go`
- `internal/testkit/envkit#FullEnv`
- `internal/battencli#intentionallyNil`
- `internal/shedbuild#engineMinimalConfig`
- `internal/loomshed#discussionWrite`
- `internal/loomshed#discussionWrite.Call`
- `internal/loomshed#TestNewDiscussionGate`
- `internal/loomcli#loomCLI.wire`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedadapters#TableSource`
- `internal/shedadapters#NewMultiLLMProducerSourced`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#requireSeam`
- `internal/shedrecipe#configRejectUnknown`
- `internal/shedrecipe#parentReviewEnv`
- `internal/shedrecipe#parentReviewCfg`
- `internal/parentreview#Store.PrepareRound`
- `internal/loomshed#NewDiscussionWrite`
- `internal/testkit/shedfake#SeatRunner`
- `internal/loomshed#NewDiscussionGate`
- `internal/loomcli#hubLocation`
- `internal/loomcli#TestWire_DiscussionSpecEvaluatesToExpectedShape`
- `internal/loomengine#DiscussionTable`
- `internal/seatengine#Prompts`

**Intent:** Register the seat-table Discussion producer under its own engine name, `DiscussionSeats`, beside the unchanged `DiscussionWrite` entry (the parent's decision superseding D1; D2, D3, D9).
`Env` gains `DiscussionTable`, a `shedadapters.TableSource` the entry evaluates once per `Call`; its doc keeps the reason it is a closure, the same reason `DiscussionSpec` gives, and `DiscussionSpec` stays for the single-agent entry.
`discussionSeatsEntry`, in a new file `entries_discussionseats.go`, follows `discussionWriteEntry`'s shape: it resolves the row's `gates` key through `resolveGateSpec` under the entry name `DiscussionSeats`, rejects every other config key, requires `DiscussionTable`, `CommitDiscussion` and `Seats` and never `Shuttle`, wraps the source so the evaluated table carries the resolved gate, wires `PrepareRound` as a `func() error` preparation exactly when the gate list holds an enabled `parent-review` entry and leaves it nil otherwise, and returns `loomshed.NewDiscussionWrite` over `NewMultiLLMProducerSourced`; its doc comment states that zero advisors is the single-agent case and why the table arrives as a closure.
Bound: a nil preparation skips nothing the single-agent row runs, since without an enabled `parent-review` gate that row wires none either.
`registry` maps `DiscussionSeats` to the entry; `TestNames` lists it; `minimalEntries` gains it over `newTestEnv` with an empty config, and `newTestEnv` and `FullEnv` fill `DiscussionTable` with a one-chair table over one absolute output path under their temp root and the chair stencil name, so the conformance test and `TestBuild_EveryRegisteredEngineBuilds` cover the new engine.
`coverageGuardAllowedUnreachableEngines` gains `DiscussionSeats` with the reason that no recipe row names it until card 8 makes loomrecipe select it for the Discussion-Write row, which removes the entry again.
battencli's `intentionallyNil` lists `DiscussionTable` as a loom-only seam; the comment of `engineMinimalConfig` names `DiscussionSeats` as the third closure-fed writer entry; the doc comments of `discussionWrite` and `discussionWrite.Call` name the wrapped producer as a `*SingleLLMProducer` or a `*MultiLLMProducer` in practice, both of which entry-check the context first and report a non-empty pointer only for Done or a gate-failed Stuck.
`TestDiscussionSeatsEntry_ConstructionFailures` is table-driven over a nil `DiscussionTable`, a nil `CommitDiscussion`, a nil `Seats`, an unknown config key and an unrecognised gate name, each error naming `DiscussionSeats` and the field or key.
`TestDiscussionSeatsEntry_Call` drives the fake seat runner, and every done chair result it scripts, whether answered by `Results`, `RunFn` or `ResumeFn`, carries a passing `GateOutcome`, as card 6 does for `TestMultiLLMEntry_ComposedTable`, so card 6's fake never evaluates the row's gate: the `discussion` gate would fail on `newTestEnv`'s absent support log and the `parent-review` gate would open a round, either of which would make a row assert the fake rather than the entry.
A done chair asserts the source was evaluated once, the table the runner saw carries the row's resolved gate by name and budget, the pointer is the chair's first output and the commit fired once; over `parentReviewEnv` and `parentReviewCfg`, a fresh spawn opens a round, a live chair scripted through `ProbeFn` with a non-nil chair handle and `ResumeFn` with a done result opens none, a disabled entry opens none, and a store whose root cannot be created makes the preparation fail so that no seat runs and the pre-written chair outputs are not archived.
The shedrecipe package doc gains a paragraph on the `DiscussionSeats` entry's told table and preparation.
In `loomcli`, `wire` fills `DiscussionTable` with a closure over `DiscussionTable(location, websterGeom.StencilsDir, loomCfg, registry, seedSlug(location.WorktreeName))`, evaluated per call for the same stencil-ownership reason the `DiscussionSpec` closure states, so the new func-typed seam is filled in the card that adds it and `TestWire_EverySeamFilled` stays green; the closure is built whatever `discussion_producer` holds, and only a row built on `DiscussionSeats` evaluates it.
`TestWire_DiscussionTableEvaluatesToExpectedShape` evaluates the closure for both `discussion_interactive` values and asserts the table's interactivity, role prefix, segment, timeout, the chair's skills, outputs and non-empty model, one advisor per template entry, and, through `seatengine.Prompts` over the wired geometry, a chair prompt naming the parent with no unrendered marker; `TestWire_DiscussionSpecEvaluatesToExpectedShape` stays as the single-agent closure's test.
A row joins `TestNewDiscussionGate` that plants an `advisor-1.md` whose content would fail the discussion checks beside a valid decision record and support log, and asserts the gate passes with no findings, pinning D7's bound that the gate reads the two discussion files only.
The budget file's `internal/shedrecipe` row rises by two and its `internal/loomcli` row by one for the new tests.

**ImpactSummary:** One Env field and one registry entry added, with every filler of a full Env, `loomcli`'s `wire` included, touched to carry the new seam; the `DiscussionWrite` entry, its tests and every recipe row are unchanged, and the new engine is reached by no row until card 8.
