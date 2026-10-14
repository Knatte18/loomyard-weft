# Card 12 — plan-write-seats

**Create:**
- `plan:internal/loomengine#PlanTable` -> `func PlanTable(layout *lyxcwd.Location, dirs configengine.Dirs, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry) (seatengine.Table, error)`
- `plan:internal/loomengine#TestPlanTable` -> `func TestPlanTable(t *testing.T)`

**Delete:**
- `internal/loomengine#TestPlanSpec`
- `internal/loomengine#TestPlanSpec_PatternDirective`
- `internal/loomengine#TestPlanSpec_Refuses`
- `internal/loomengine#TestPlanSpec_Twill`
- `internal/loomengine#TestCompose_FrictionDirective`

**Edit:**
- `internal/loomengine#planTwillDirective`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/shedrecipe#planWriteEntry`
- `internal/shedrecipe#planWriteGateSpec`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#TestPlanWriteEntry_ConstructionFailures`
- `internal/shedrecipe#TestPlanWriteEntry_HappyPath`
- `internal/shedrecipe#TestPlanWriteEntry_CallDone`
- `internal/shedrecipe#TestPlanWriteEntry_CallAppendsPriorPlanBlock`
- `internal/shedrecipe#TestPlanWriteEntry_GateConfig`
- `internal/loomshed#planWrite`
- `internal/loomshed#planWrite.Call`
- `internal/loomshed#NewPlanWrite`
- `internal/loomshed#NewPlanDirRotator`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#FullEnv`

**Intent:** D9 and D10 for Plan-Write: the row runs on `MultiLLMProducer` over a one-seat table.

`loomengine.PlanTable` takes over from `PlanSpec` and its composer, which no row calls any more and card 18 deletes: one lead seat opening with `loom-template-plan` (which includes no seat directive), role prefix `plan`, the plan role's model, effort and version, the overview as its one output, `planSkills`, the plan segment, `plan_timeout_min`, not interactive.
Its table values are the stencil's markers (`decision_record_path`, `plan_dir`, `overview_path`, `specs_dir`, and the optional `pattern_directive` and friction directive, named in `Optional`); the seat engine renders the parent and edit directives (PATTERN-parent-directive, PATTERN-edit-directive), so the builder takes no parent name.
Lens forks keep today's switch until card 19: under `plan_producer: twill` the lead sets `ForkSubagents` and carries `setOffParagraph(planTwillDirective(...))` as its own seat value; otherwise neither, and the prompt keeps its bytes. An unresolvable `twill-plan` fan is refused at table build as today.
The lead's output files equal the old spec's, so the seat probe, which shuttle matches by output files, finds a Plan-Write run `SingleLLMProducer` started before the deploy and attaches to it instead of respawning.
`plan.go`'s file header describes the table once card 18 removes the spec builder it describes today.

`planWriteEntry` builds `shedadapters.NewMultiLLMProducerSourced(name, tables, env.Seats, env.Now, rotate)` behind `loomshed.NewPlanWrite`, where `tables` stamps the `planWriteGateSpec` gate on each table `Env.PlanTable` builds and `rotate` is `loomshed.NewPlanDirRotator`, whose amendment the producer appends to the lead's prompt on the fresh path only (cards 9 and 10); it requires `PlanTable`, `CommitPlan`, `Seats` and the roots it requires today, and never reads `Shuttle`; its doc comment and `entries_planwrite.go`'s file header name the seat producer.
`planWriteGateSpec`'s comment says the forked gate applies to a run whose lead carries fork authorization.
`Env.PlanSpec` becomes `Env.PlanTable`, a `shedadapters.TableSource`; `loomCLI.wire` fills it from `PlanTable` with `fabricengine.ConfigDirs(location)`.
`loomshed`'s `planWrite`, `planWrite.Call`, `NewPlanWrite` and `NewPlanDirRotator` comments, and `planwrite.go`'s header, name the seat producer and its lead in place of `SingleLLMProducer`.

Tests:
- `TestPlanTable` replaces the four `TestPlanSpec*` tests and the plan row of `TestCompose_FrictionDirective`, which then has no row left and goes. Its parity rows build both `PlanSpec` (still present until card 18) and `PlanTable` over one config, with and without twill, and assert the lead's role (the prefix), model, effort, version, output files, skills, segment, timeout, interactivity and fork flag equal the spec's, and the lead's prompt from `seatengine.Prompts` (under a geometry naming the same parent) is byte-equal to the spec's prompt, so the seat probe matches a run the old spec started and the prompt keeps its bytes; under twill the fork flag is on and the lead's prompt (`seatengine.Prompts`) holds the twill directive between Step 2 and Step 3 with the plan host's values; the implementer PATTERN directive and a resolved friction note path land in the prompt, an empty friction directive renders nothing; an unparsable plan model-spec, a missing stencil and an unresolvable `twill-plan` fan are refused.
- `TestProducerSpecs_SkillsAndParentDirective`'s plan row builds from `PlanTable`.
- `TestPlanWriteEntry_*` drive the entry over `shedfake.SeatRunner`: the missing-seam rows name `PlanTable` and `Seats`; `CallDone` asserts one table ran with the gate and the commit ran once; `CallAppendsPriorPlanBlock` asserts the table the runner ran carries the rendered prior-plan block as its `Amendment`, and a new resume row (a live lead in the runner's `LiveTables`, and a done lead with the overview as its output in its `ResumeResults`) asserts the stale plan file stays in place, no table ran and the resumed table carries no amendment.
- `newTestEnv`, `envkit.FullEnv`, `buildSequenceFixture` (an `env.PlanTable` whose lead writes the overview, served by `newLoomSeats`' `plan` branch) and `TestWire_DefaultConfig` (evaluates `c.env.PlanTable()`) follow the field; `battencli`'s `intentionallyNil` renames its `PlanSpec` key.

**ImpactSummary:** Plan-Write's producer changes for every hub with its strand role, outputs and gate unchanged; Env.PlanSpec leaves every caller listed here and PlanSpec is left uncalled.
