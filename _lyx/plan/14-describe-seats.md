# Card 14 — describe-seats

**Create:**
- `plan:internal/landingshed#DescribeTable` -> `func DescribeTable(in DescribeInputs, cfg Config, reg modelspec.Registry) (seatengine.Table, error)`
- `plan:internal/landingshed#DarnDescribeTable` -> `func DarnDescribeTable(in DarnDescribeInputs, cfg Config, reg modelspec.Registry) (seatengine.Table, error)`
- `plan:internal/landingshed#describeSessionTable` -> `func describeSessionTable(stencilName, descriptionPath string, markers map[string]string, cfg Config, reg modelspec.Registry) (seatengine.Table, error)`

**Edit:**
- `internal/landingshed#newDescribeInputs`
- `internal/landingshed#TestDescribeSpec_ComposesSpec`
- `internal/landingshed#TestDescribeSpec_SkillsAndParentDirective`
- `internal/landingshed#TestDescribeSpec_RunRecordsOldestFirstLiveLast`
- `internal/landingshed#TestDescribeSpec_EmptyFieldIsError`
- `internal/landingshed#TestDarnDescribeSpec`
- `internal/landingshed#TestDescribeSpec_UnparsableModelSpecIsError`
- `internal/shedrecipe#describeEntry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#newDescribeTestEnv`
- `internal/shedrecipe#TestDescribeEntry_ConstructionFailures`
- `internal/shedrecipe#TestDescribeEntry_HappyPath`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#FullEnv`

**Intent:** D10 for Describe, loom's and darn's: the row runs on `MultiLLMProducer` over a one-seat table.

`landingshed.DescribeTable` and `DarnDescribeTable` take over from the two spec builders, which no row calls any more and card 18 deletes with `describeSessionSpec`, keeping their required-field refusals by name, their markers and `describeSessionTable` as the shared half: one lead seat opening with the told stencil (no seat directive), role prefix `describe`, the describe model, effort and version, the description as its one output, `describeSkills`, the describe segment, `describe_timeout_min`, not interactive, no fork flag, with the markers as table values.
The seat engine renders the parent and edit directives, so the table builders read no `ParentName` (the field leaves `DescribeInputs` and `DarnDescribeInputs` in card 18, with the spec builders that read it); `StencilsDir` stays a required field, since the builder reads the stencil to refuse a missing one before the row runs.
The lead's output file equals the old spec's, so a Describe run `SingleLLMProducer` started before the deploy is attached, not respawned.
`describe.go`'s file header describes the table builders once card 18 removes the spec builders.

`describeEntry` builds `shedadapters.NewMultiLLMProducerSourced(name, tables, env.Seats, env.Now, nil)` over `Env.DescribeTable` with the row's gate stamped on, behind `loomshed.NewDiscussionWrite`'s commit decorator; it requires `DescribeTable`, `CommitDescription` and `Seats`; its doc comment and `entries_describe.go`'s file header name the seat producer.
`Env.DescribeSpec` becomes `Env.DescribeTable`, a `shedadapters.TableSource`; `loomCLI.wire` and `loomCLI.wireDarn` fill it from `DescribeTable` and `DarnDescribeTable`, opening the fabric lazily as today.

Tests: the six describe tests and `newDescribeInputs` build tables; each asserts on the lead seat and table fields and on the lead's prompt through `seatengine.Prompts` under a geometry naming the parent (the parent and no-parent variants in `TestDescribeSpec_SkillsAndParentDirective`); `TestDescribeSpec_ComposesSpec` and `TestDarnDescribeSpec` build the old spec beside the table (both present until card 18) and assert the lead's role, model, effort, version, output file, skills, segment, timeout, interactivity and fork flag equal the spec's and the lead's prompt is byte-equal to the spec's; the run-record order, the empty-field refusals and the unparsable model-spec refusal keep their rows; `TestDarnDescribeSpec` keeps its darn markers.
`TestDescribeEntry_*` and `newDescribeTestEnv` drive the entry over `shedfake.SeatRunner`, the missing-seam rows naming `DescribeTable` and `Seats`, and `TestDescribeEntry_HappyPath` calls the producer once and asserts the table the runner ran carries the row's resolved `description` gate; `buildSequenceFixture` sets an `env.DescribeTable` served by `newLoomSeats`' `describe` branch; `envkit.FullEnv` follows the field and drops its local `specOver`, which fills no field any more; `battencli`'s `intentionallyNil` renames its `DescribeSpec` key.

**ImpactSummary:** Both Describe recipes' producer changes with role, output and gate unchanged; Env.DescribeSpec leaves every caller listed here, and DescribeSpec and DarnDescribeSpec are left uncalled.
