# Card 15 — darn-write-seats

**Create:**
- `internal/darnengine#DarnTable`

**Edit:**
- `internal/darnengine/doc.go`
- `internal/darnengine#newSpecInputs`
- `internal/darnengine#TestDarnSpec`
- `internal/darnengine#TestDarnSpec_Refusals`
- `internal/shedrecipe#darnWriteEntry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#darnTestEnv`
- `internal/shedrecipe#TestDarnWriteEntry_ConstructionFailures`
- `internal/shedrecipe#TestDarnWriteEntry_HappyPath`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#intentionallyNil`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/battencli#intentionallyNil`

**Intent:** D10 for Darn-Write: the row runs on `MultiLLMProducer` over a one-seat table.

`darnengine.DarnTable` takes over from `DarnSpec`, which no row calls any more and card 18 deletes, keeping its required-field refusals by name and its writer model-spec refusal: one lead seat opening with `darn-template-write` (no seat directive), role prefix `darn`, the writer's model, effort and version, the description as its one output, `darnSkills`, the darn spec's segment, `writer_timeout_min`, not interactive, no fork flag.
Its table values are the stencil's markers (`slug`, `description_path`, `specs_dir`, `rejection_findings` and `prior_work`, each of the last two `(none)` when empty, and the optional `pattern_directive` and friction directive); the seat engine renders the parent and edit directives, so `DarnTable` reads no `ParentName` (the field leaves `DarnInputs` in card 18, with `DarnSpec`).
The lead's output file equals the old spec's, so a Darn-Write run `SingleLLMProducer` started before the deploy is attached, not respawned.
`doc.go` describes the table; `spec.go`'s file header follows in card 18.

`darnWriteEntry`'s per-session producer becomes `shedadapters.NewMultiLLMProducerSourced(name, tables, env.Seats, env.Now, nil)` over `Env.DarnTable(told)` with the row's gate stamped on, behind `loomshed.NewDarnWrite`; it requires `DarnTable` and `Seats` with the `Darn` seams it requires today; its doc comment and `entries_darnwrite.go`'s file header name the seat producer.
`Env.DarnSpec` becomes `Env.DarnTable`, `func(loomshed.DarnTold) (seatengine.Table, error)`; `loomCLI.wireDarn` fills it from `DarnTable`.

Tests: `TestDarnSpec` and `TestDarnSpec_Refusals` (with `newSpecInputs`) build tables and assert the lead seat and table fields; `TestDarnSpec` builds `DarnSpec` (present until card 18) beside the table and asserts the lead's role, model, effort, version, output file, skills, segment, timeout, interactivity and fork flag equal the spec's and the lead's prompt is byte-equal to the spec's; it asserts the rendered tokens of a first and a later spawn through `seatengine.Prompts`, and the parent and no-parent variants under a geometry; the refusal rows keep their field names.
`TestDarnWriteEntry_*` and `darnTestEnv` drive the entry over `shedfake.SeatRunner`, the missing-seam rows naming `DarnTable` and `Seats`, and `TestDarnWriteEntry_HappyPath` calls the producer once and asserts the table the runner ran carries the row's resolved gate.
`TestWire_DarnRecipe`'s darn-seam list and its loom-seed assertion name `DarnTable`; both `intentionallyNil` maps rename their `DarnSpec` key.

**ImpactSummary:** Darn-Write's producer changes with role, output and gate unchanged; Env.DarnSpec leaves every caller listed here and DarnSpec is left uncalled.
