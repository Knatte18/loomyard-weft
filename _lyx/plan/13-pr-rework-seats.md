# Card 13 — pr-rework-seats

**Create:**
- `plan:internal/loomengine#ReworkTable` -> `func ReworkTable(layout *lyxcwd.Location, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry, firstCard int, priorPlanDir string) (seatengine.Table, error)`
- `plan:internal/loomengine#TestReworkTable` -> `func TestReworkTable(t *testing.T)`

**Delete:**
- `internal/loomengine#TestReworkSpec`
- `internal/loomengine#TestReworkSpec_MissingStencil`

**Edit:**
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/shedrecipe#prReworkEntry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#reworkTestEnv`
- `internal/shedrecipe#TestPRReworkEntry_HappyPath`
- `internal/shedrecipe#TestPRReworkEntry_EvaluatesReworkSpecWithToldValue`
- `internal/shedrecipe#TestPRReworkEntry_ConstructionFailures`
- `internal/shedrecipe#TestPRReworkEntry_Config`
- `internal/loomcli#loomCLI.wire`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#FullEnv`

**Intent:** D10 for PR-Rework: the row runs on `MultiLLMProducer` over a one-seat table.

`loomengine.ReworkTable` takes over from `ReworkSpec` and its composer, which no row calls any more and card 18 deletes: one lead seat opening with `loom-template-rework` (no seat directive), role prefix `rework`, `ResolveRework`'s model, effort and version, the coverage file as its one output, `reworkSkills`, the webster segment, `plan_timeout_min`, not interactive, no fork flag.
It builds its paths through the existing `reworkPaths` struct, which outlives `composeReworkPrompt`. Its table values are the stencil's markers (the rejection, plan, overview, decision record, coverage, plan stencil and prior plan paths, `first_card` and `specs_dir`, with `pattern_directive` and the friction directive optional); the seat engine renders the parent and edit directives.
The lead's output files equal the old spec's, so a PR-Rework run `SingleLLMProducer` started before the deploy is attached, not respawned.
`rework.go`'s file header describes the table once card 18 removes the spec builder it describes today.

`prReworkEntry`'s per-session producer becomes `shedadapters.NewMultiLLMProducerSourced(name, tables, env.Seats, env.Now, nil)` over `Env.ReworkTable(told)` with the row's gate stamped on, behind `loomshed.NewPRRework`; it requires `ReworkTable` and `Seats` with the seams and roots it requires today, and never reads `Shuttle`; its doc comment and `entries_prrework.go`'s file header name the seat producer.
`Env.ReworkSpec` becomes `Env.ReworkTable`, `func(loomshed.ReworkTold) (seatengine.Table, error)`; `loomCLI.wire` fills it from `ReworkTable`.

Tests:
- `TestReworkTable` replaces the two `TestReworkSpec*` tests: built beside `ReworkSpec` (present until card 18) over one config, the lead's role, model (the rework key, falling back to plan), effort, version, output file, skills, segment, timeout, interactivity and fork flag equal the spec's, and the lead's prompt from `seatengine.Prompts` under a geometry naming the same parent is byte-equal to the spec's prompt; the prompt holds every told path, the first card and the prior plan directory; a missing stencil is refused.
- `TestProducerSpecs_SkillsAndParentDirective`'s rework row builds from `ReworkTable`, so every row of it now reads a table.
- `TestPRReworkEntry_*` and `reworkTestEnv` drive the entry over `shedfake.SeatRunner`: the told value reaches `ReworkTable`, `TestPRReworkEntry_Config` asserts the table the runner ran (`GotTables[0].Gate`) carries the row's resolved gate, the missing-seam rows name `ReworkTable` and `Seats`.
- `envkit.FullEnv` follows the field; `battencli`'s `intentionallyNil` renames its `ReworkSpec` key.

**ImpactSummary:** PR-Rework's producer changes with its strand role, output and gate unchanged; Env.ReworkSpec leaves every caller listed here and ReworkSpec is left uncalled.
