# Card 8 — discussion-write-row-selection

**Create:**
- `plan:internal/shedbuild#NewShedFrom` -> `func NewShedFrom(recipe Recipe, env shedrecipe.Env, paths ShedPaths) (*shedengine.Shed, error)`
- `plan:internal/loomrecipe#discussionSeatsEngine` -> `const discussionSeatsEngine = "DiscussionSeats"`
- `plan:internal/loomrecipe#applyDiscussionProducer` -> `func applyDiscussionProducer(recipe *shedbuild.Recipe, seats bool)`
- `plan:internal/loomrecipe#newLoomSeats` -> `func newLoomSeats(writeOutputs bool) *shedfake.SeatRunner`
- `plan:internal/loomrecipe#TestNew_DiscussionSeatsRunsTheWriterRowOnSeats` -> `func TestNew_DiscussionSeatsRunsTheWriterRowOnSeats(t *testing.T)`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`
- `internal/shedbuild#NewShed`
- `internal/shedbuild#TestNewShed_FieldCompleteAssembly`
- `internal/loomrecipe#New`
- `internal/loomrecipe#RecipeEngines`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `internal/loomrecipe#gatedRowFanViolations`
- `internal/loomrecipe#TestNoGatedRowAuthorizesForkSubagents`
- `internal/loomrecipe/doc.go`
- `internal/loomcli#loomCLI.wire`
- `plan:internal/loomcli#TestWire_DiscussionTableEvaluatesToExpectedShape`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedbuild#Parse`
- `internal/shedbuild#Build`
- `internal/shedbuild#Recipe`
- `internal/loomshed#NameDiscussionWrite`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/loomrecipe#newLoomShuttle`
- `internal/loomrecipe#countRole`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomcli#hubLocation`
- `internal/loomcli#seedLoomConfigWithKeys`
- `plan:internal/loomengine#DiscussionProducerSeats`
- `internal/testkit/shedfake#SeatRunner`

**Intent:** Let `loom.yaml`'s `discussion_producer` select which of the two producers the Discussion-Write row runs, keeping the row name, its gates and its routing (the parent's decision superseding D1).
`Env` gains `DiscussionSeats bool`, a run-wide value read by loomrecipe alone, as `ReviewMaxBounces` is: true runs the Discussion-Write row on the `DiscussionSeats` engine instead of the recipe's `DiscussionWrite`.
`NewShedFrom` builds a parsed `Recipe` against an env and assembles the shed with the paths' fields, and `NewShed` becomes `Parse` followed by `NewShedFrom`, so a caller can change a parsed recipe before it is built; a row joins `TestNewShed_FieldCompleteAssembly` pinning that the two assemble the same shed from the same bytes.
`applyDiscussionProducer` sets the engine of the row named `loomshed.NameDiscussionWrite` to `discussionSeatsEngine` when `seats` is true and leaves the recipe alone otherwise; `New` parses the recipe, applies it from `env.DiscussionSeats` and builds through `NewShedFrom`, after its existing coherence and budget checks, so routing, the row name and the status file are unchanged whichever engine the row builds with, and `Routing` never reads the choice.
Bound: the substitution happens after the parse, so the built row escapes `TestNoGatedRowAuthorizesForkSubagents`, which reads only the parsed recipe file, and `DiscussionSeats` is not among `gatedRowFanViolations`' writer engines; the substituted row keeps the Discussion-Write row's gates on the chair, and the seat engine builds every seat's spec without `ForkSubagents`, so the chair never forks while its gate runs.
`gatedRowFanViolations` adds `DiscussionSeats` to its writer-engine case, its message and its doc's writer list, and `TestNoGatedRowAuthorizesForkSubagents` gains a row running the guard over the shipped recipe after `applyDiscussionProducer(recipe, true)`, with the Discussion-Review and Plan-Review fans, expecting no violation, so the guard covers the substituted recipe too.
`RecipeEngines` returns the recipe's engines plus `discussionSeatsEngine`, still sorted and de-duplicated, since the substitution makes it an engine a loom row reaches, and its doc says so; `coverageGuardAllowedUnreachableEngines` drops the entry card 7 added, which the guard's drift check now requires.
`TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` derives its want set from `wantProducerTable`'s engine column plus `discussionSeatsEngine`, sorted, so it still catches a silently empty or shrunk return while `wantProducerTable` keeps describing the default build.
The loomrecipe package doc names the selection beside the review budget as the second value `New` applies from the env.
`newLoomSeats` returns a fake seat runner whose `RunFn` writes the chair's outputs with the fixture's valid discussion content when told to and reports a done chair with no `GateOutcome`, so card 6's fake evaluates Discussion-Write's own gate; `TestNew_DiscussionSeatsRunsTheWriterRowOnSeats` builds the sequence fixture with `DiscussionSeats` true, `DiscussionTable` returning a chair over the fixture's two discussion paths and `Seats` from `newLoomSeats(true)`, runs the shed with row 1 substituted, and asserts the Discussion-Write history entry is Done, the seat fake saw exactly one table carrying the row's `discussion` and `parent-review` gates, the loom shuttle ran no `discussion` role, and the commit closure fired; a second subtest with `DiscussionSeats` false asserts only that the seat fake saw no table, since `TestNew_ShapeMatchesRecipe` and the existing sequence tests already pin the default build's shape and its `discussion` shuttle role.
In `loomcli`, `wire` fills `DiscussionSeats` with whether `loomCfg.DiscussionProducer` equals `DiscussionProducerSeats`; the `DiscussionTable` closure is card 7's.
`TestWire_DiscussionTableEvaluatesToExpectedShape` gains a subtest that seeds `discussion_producer: seats` through `seedLoomConfigWithKeys` and asserts `DiscussionSeats` is true then and false under the template.
The budget file's `internal/loomrecipe` row rises by one.

**ImpactSummary:** One Env field, one shedbuild seam split out of `NewShed` with its callers unchanged, and one pre-build substitution in `New` that fires only when the env asks for it; every existing loomrecipe and loomcli test keeps its single-agent shape, the engine-set test and the gate-quiescence guard gain the one substituted engine, and the recipe file is unchanged.
