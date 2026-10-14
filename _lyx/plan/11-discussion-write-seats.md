# Card 11 — discussion-write-seats

**Create:**
- `plan:internal/loomrecipe#writeLoomOutputs` -> `func writeLoomOutputs(planDir string, writeOutputs bool, role string, outputs []string) error`
- `plan:internal/loomrecipe#countSeatRole` -> `func countSeatRole(seats *shedfake.SeatRunner, prefix string) int`
- `plan:internal/loomcli#scriptedSeats` -> `type scriptedSeats struct`
- `plan:internal/loomcli#scriptedSeats.Probe` -> `func (s *scriptedSeats) Probe(table seatengine.Table) (seatengine.LiveTable, error)`
- `plan:internal/loomcli#scriptedSeats.Resume` -> `func (s *scriptedSeats) Resume(table seatengine.Table, live seatengine.LiveTable) (seatengine.Result, error)`
- `plan:internal/loomcli#scriptedSeats.Run` -> `func (s *scriptedSeats) Run(table seatengine.Table) (seatengine.Result, error)`

**Edit:**
- `internal/loomengine#DiscussionTable` -> `func DiscussionTable(layout *lyxcwd.Location, dirs configengine.Dirs, stencilsDir string, cfg Config, reg modelspec.Registry, slug string) (seatengine.Table, error)`
- `internal/loomengine#discussionTwillDirective`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestDiscussionTable_Refuses`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/loomengine#TestCompose_FrictionDirective`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#registry`
- `internal/shedrecipe/doc.go`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#TestDiscussionWriteEntry_ConstructionFailures`
- `internal/shedrecipe#TestDiscussionWriteEntry_HappyPath`
- `internal/shedrecipe#TestDiscussionWriteEntry_CallDone`
- `internal/shedrecipe#TestDiscussionWriteEntry_GateConfig`
- `internal/shedrecipe#TestDiscussionWriteEntry_ParentReviewBeginRound`
- `internal/shedrecipe#hasRound`
- `internal/loomrecipe#New`
- `internal/loomrecipe#RecipeEngines`
- `internal/loomrecipe/doc.go`
- `internal/loomrecipe#newLoomSeats` -> `func newLoomSeats(planDir string, writeOutputs bool) *shedfake.SeatRunner`
- `internal/loomrecipe#runLoomShuttle`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `internal/loomrecipe#TestNoGatedRowAuthorizesForkSubagents`
- `internal/loomrecipe#gatedRowFanViolations`
- `internal/loomrecipe#TestResume_DiscussionWriteRespawnsRatherThanReportDoneOffFileExistence`
- `internal/loomrecipe#TestResume_LiveMatchingRunAttachesInsteadOfRespawning`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_DiscussionTableEvaluatesToExpectedShape`
- `internal/loomcli#prFixture`
- `internal/loomcli#newPRFixture`
- `internal/loomcli#TestParentReviewExchange_RewriteReviewedThenCapRejectHalts`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#FullEnv`
- `internal/shedbuild#engineMinimalConfig`
- `contracts/stencils#roleOpeningStencils`
- `//contracts/stencils/loom/loom-template-discussion.md`
- `//contracts/stencils/loom/loom-template-discussion-chair.md`
- `//contracts/specs/shed-recipe-spec.md`
- `//docs/overview.md`

**Delete:**
- `internal/loomengine#TestDiscussionSpec`
- `internal/loomengine#TestDiscussionSpec_Refuses`
- `internal/loomengine#TestDiscussionSpec_Twill`
- `internal/loomengine#TestDiscussionSpec_ReadsStencilAtCallTime`
- `internal/loomengine#TestDiscussionSpec_PatternDirective`
- `internal/loomengine#TestComposePrompt_RendersMarkers`
- `//internal/shedrecipe/entries_discussionseats.go`
- `//internal/shedrecipe/entries_discussionseats_test.go`
- `internal/loomrecipe#applyDiscussionProducer`
- `internal/loomrecipe#discussionSeatsEngine`
- `internal/loomrecipe#TestNew_DiscussionSeatsRunsTheWriterRowOnSeats`
- `internal/loomcli#TestWire_DiscussionSpecEvaluatesToExpectedShape`
- `internal/loomcli#scriptedShuttle`
- `internal/loomcli#scriptedShuttle.Run`
- `internal/loomcli#scriptedShuttle.Attach`
- `internal/loomcli#scriptedShuttle.AttachIfLive`
- `internal/loomcli#scriptedShuttle.AttachGated`
- `internal/loomcli#scriptedShuttle.RunGated`

**Intent:** D8 and D10 for Discussion-Write: the row always runs on `MultiLLMProducer` over a seat table, so the `DiscussionSeats` engine swap and the single-writer spec go.

`loomengine.DiscussionTable` becomes the row's only builder, keeping today's `discussion_producer` meaning until card 19 retires the key:
- The lead opens with `loom-template-discussion-chair`, the stencil that wraps the single writer's prompt and the chair's seat directive, whose text covers a lead with no advisors (D8).
- Advisors: one per `discussion_advisors` entry, and only when `discussion_producer` is `seats`; under `single` or `twill` the table has the lead alone, so the hub's dormant advisor list never starts one.
- Lens forks: under `twill` the lead alone sets `ForkSubagents` and carries the twill directive as its own seat value, set off as a paragraph (`setOffParagraph(discussionTwillDirective(...))`), read through the new `dirs` argument; no advisor seat sets `ForkSubagents` or carries the directive (D8).
  A fan that does not resolve is refused at table build, naming the fan and `burler.yaml`, the refusal `discussionTwillDirective` already makes (its `refusal-spec.md` rows stand).
- The lead keeps the single writer's role (`discussion`, the prefix), model, effort, version, output files (the decision record and the support log), interactivity (`discussion_interactive`, the same setting both producers read), skills, segment, timeout and fork flag; its prompt changes only by the seat directive and the advisors section the chair stencil adds.
No row calls `DiscussionSpec` or its prompt composer `composePrompt` any more, and their tests go here; card 18 deletes the two once no later card edits code that names them.

In `internal/shedrecipe`, `discussionWriteEntry` builds `shedadapters.NewMultiLLMProducerSourced(name, tables, env.Seats, env.Now, prepare)` behind `loomshed.NewDiscussionWrite`, where `tables` stamps the row's resolved gate on each table `Env.DiscussionTable` builds and `prepare` is the parent-review round preparation (returning an empty amendment) when the row's gate list enables `parent-review`; it requires `DiscussionTable`, `CommitDiscussion` and `Seats`, and never reads `Shuttle`; its doc comment and `entries_discussionwrite.go`'s file header name the seat producer.
`Env` loses `DiscussionSpec` and `DiscussionSeats`, and `DiscussionTable`'s comment names the `DiscussionWrite` entry; `discussionSeatsEntry`, its file and its test file, and the `DiscussionSeats` registry row go (PATTERN-shed-recipe-registry); `doc.go`'s DiscussionSeats section is folded into the DiscussionWrite description.
`internal/loomrecipe` loses `applyDiscussionProducer` and `discussionSeatsEngine`; `New` builds the parsed recipe as written, `RecipeEngines` derives the set from the two recipes alone, and `doc.go` drops the substitution paragraph.
`loomCLI.wire` drops the `DiscussionSpec` closure and the `DiscussionSeats` flag, and passes `fabricengine.ConfigDirs(location)` to `DiscussionTable`.
`roleOpeningStencils` maps `discussion` to `loom-template-discussion-chair`, drops `discussion-chair`, and keeps `discussion-advisor` as the nominal key of the advisor stencil (D6), so PATTERN-parent-directive and PATTERN-edit-directive keep checking the lead's opening stencil.
The two discussion stencils' header comments (including the discussion stencil's line saying `loomengine.DiscussionSpec` wraps the filled result into a spec), `shed-recipe-spec.md`'s Discussion-Write paragraph and `docs/overview.md`'s Discussion paragraph describe the one seat-table producer.

Tests:
- `TestDiscussionTable` rows, comparing the lead seat and table fields the seat engine maps one to one into the lead's spec (`TestSeatPrompts_ValuesPromptsAndSpecs` pins that mapping) against `DiscussionSpec`'s answer over the same config, which stays until card 18: `single` builds a lone lead with role `discussion`, the old spec's model, effort, version, output files, skills, segment, timeout and interactivity, and no fork flag; `twill` the same with `ForkSubagents` and the twill directive on the lead only; `seats` with advisors builds the advisors with neither; seats with advisors and lenses is not reachable before card 19. The lead's prompt (`seatengine.Prompts`) holds the discussion stencil's text, the PATTERN directive, and the chair directive; the twill rows of the deleted `TestDiscussionSpec_Twill` (template fan, a `burler.yaml` redefining the fan, the fan only in the board default) move here.
- `TestDiscussionTable_Refuses` gains the unresolvable-fan row under `twill`.
- `TestProducerSpecs_SkillsAndParentDirective` builds its discussion rows from `DiscussionTable` and reads skills and the prompt through `seatengine.Prompts` under a geometry naming the parent; the plan and rework rows stay on their specs until cards 12 and 13.
- `TestCompose_FrictionDirective` drops its discussion composer row; the friction path in the discussion lead's prompt is asserted in `TestDiscussionTable`.
- `TestDiscussionWriteEntry_*` and `hasRound` drive the entry over `shedfake.SeatRunner`: the missing-seam rows name `DiscussionTable` and `Seats`; `CallDone` asserts the runner received the table with the row's gate and the commit ran once; `ParentReviewBeginRound` covers the enabled and disabled parent-review preparation the deleted `TestDiscussionSeatsEntry_Call` covered.
- `minimalEntries`, `TestNames` and `shedbuild`'s `engineMinimalConfig` (its entry and the doc comment naming `DiscussionSeats` and `DiscussionSpec`) drop `DiscussionSeats`; `newTestEnv` and `envkit.FullEnv` drop `DiscussionSpec`; `battencli`'s `intentionallyNil` drops its `DiscussionSpec` key.
- `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`, `TestNoGatedRowAuthorizesForkSubagents` and `gatedRowFanViolations` (with its comment) drop the substituted engine.
- `writeLoomOutputs` holds `runLoomShuttle`'s per-role writes, keyed by role; `runLoomShuttle` calls it with the spec's role and outputs, and `newLoomSeats` with the table's role prefix and the lead's outputs, so one fixture serves a row on either producer. `buildSequenceFixture` sets `env.Seats = newLoomSeats(planDir, true)` and an `env.DiscussionTable` whose lead writes the two discussion files. `countSeatRole` counts the tables of one prefix the fake ran.
- `TestResume_DiscussionWriteRespawnsRatherThanReportDoneOffFileExistence` asserts the seat runner was probed and ran a `discussion` table; `TestResume_LiveMatchingRunAttachesInsteadOfRespawning` sets a `ProbeFn` on the fixture's runner answering a live lead (a non-nil handle) for the `discussion` prefix only, every other prefix not live so later rows still run and write their outputs, and a `ResumeFn` reporting a done lead with the decision record and support log as its outputs, and asserts the runner resumed the `discussion` table, ran none and archived nothing.
- `TestNew_DiscussionSeatsRunsTheWriterRowOnSeats` goes, since `TestSequence_FullRunBlocksAtMergeIn` now runs the row on seats; `TestWire_DiscussionSpecEvaluatesToExpectedShape` goes, and `TestWire_DiscussionTableEvaluatesToExpectedShape` drops its `DiscussionSeats` assertion and asserts the wired table's lead under each `discussion_producer` value.
- `scriptedSeats` replaces `scriptedShuttle` in the parent-review integration fixture: `Probe` finds no live lead, `Run` runs the script over the table's gate and reports the lead's outputs; `prFixture`, `newPRFixture` and `TestParentReviewExchange_RewriteReviewedThenCapRejectHalts` set `env.Seats` and `env.DiscussionTable` in place of `env.Shuttle` and `env.DiscussionSpec`.

**ImpactSummary:** The Discussion-Write row's producer changes for every hub; strand role stays discussion; the DiscussionSeats engine, Env.DiscussionSpec and Env.DiscussionSeats leave every caller listed here, and DiscussionSpec is left uncalled.
