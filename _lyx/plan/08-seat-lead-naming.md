# Card 8 — seat-lead-naming

**Create:**
- `plan:internal/seatengine#ValidateSeatName` -> `func ValidateSeatName(name string) error`

**Edit:**
- `internal/seatengine#SeatRole`
- `internal/seatengine#Seat`
- `internal/seatengine#Table`
- `internal/seatengine#Table.Chair`
- `internal/seatengine#Table.Validate`
- `internal/seatengine#validateSeatRoles`
- `internal/seatengine#Table.validateNames`
- `internal/seatengine#Table.validateOutputs`
- `internal/seatengine#Table.validateRoles`
- `internal/seatengine#strandNames`
- `internal/seatengine#seatValues`
- `internal/seatengine#seatSpec`
- `internal/seatengine#seatGate`
- `internal/seatengine#Engine.Run`
- `internal/seatengine#Engine.startSeat`
- `internal/seatengine#Engine.Probe`
- `internal/seatengine#Engine.Resume`
- `internal/seatengine/doc.go`
- `internal/seatengine#validTable`
- `internal/seatengine#TestTable_Validate`
- `internal/seatengine#TestTable_NamesAccessorsAndReservedMarkers`
- `internal/seatengine#engineTable`
- `internal/seatengine#role`
- `internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs`
- `internal/seatengine#TestEngine_Run_ChairEndStopsEverySeat`
- `internal/seatengine#TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat`
- `internal/seatengine#TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair`
- `internal/seatengine#TestEngine_Run_AdvisorDeathSendsOneNoticeOnceTheChairTakesIt`
- `internal/seatengine#TestEngine_Run_AdvisorsEndingTogetherOrAsTheyStart`
- `internal/seatengine#TestEngine_Run_DoneAdvisorSendsNothingAndIsKeptUntilTheChairEnds`
- `internal/seatengine#TestEngine_Run_FailedStopReturnsTheWayForwardUnretried`
- `internal/seatengine#TestEngine_Run_RequiredAdvisorEndsTheStep`
- `internal/seatengine#TestEngine_Run_ChairHoldDeferral`
- `internal/seatengine#TestEngine_ProbeAndResume_AttachToALiveChair`
- `internal/seatengine#TestEngine_Probe_ChairNotLiveStopsTheLiveAdvisorsAndLeavesAFreshStart`
- `internal/seatengine#TestEngine_ProbeAndResume_Refusals`
- `internal/seatengine_test#TestSmokeSeatsChairAsksAdvisor`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#Profile.fixTable`
- `internal/burlerengine/doc.go`
- `internal/burlerengine#reviewHalfRole`
- `internal/burlerengine#fixHalfRole`
- `internal/burlerengine#chairPrompt`
- `internal/burlercli#Command`
- `internal/burlercli#burlerCLI.runCmd`
- `internal/burlercli/doc.go`
- `internal/burlercli#reviewChairRole`
- `internal/burlercli#fixChairRole`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#DiscussionWriterRole` -> `func DiscussionWriterRole() string`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomcli#newParentReviewConfig`
- `internal/loomcli#TestNewParentReviewConfig_KeepAliveInterval`
- `internal/loomcli#TestWire_DiscussionTableEvaluatesToExpectedShape`
- `internal/loomcli#TestSmokeBurlerReview_AttachesToALiveReviewerInsteadOfRespawning`
- `internal/shedrecipe#multiLLMSeat`
- `internal/shedrecipe#multiLLMEntry`
- `internal/shedrecipe#validMultiLLMConfig`
- `internal/shedrecipe#TestMultiLLMEntry_ConstructionFailures`
- `internal/shedrecipe#TestMultiLLMEntry_ComposedTable`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#withSeatTable`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe_test#TestMultiLLMFixtureRecipe_BuildsAndRunsAgainstFakeSeats`
- `internal/shedrecipe/testdata/multillm-recipe.yaml`
- `internal/shedadapters#multiLLMTable`
- `internal/loomrecipe#TestNew_DiscussionSeatsRunsTheWriterRowOnSeats`
- `internal/testkit/envkit#FullEnv`
- `contracts/stencils#roleOpeningStencils`
- `//contracts/specs/shed-recipe-spec.md`
- `//contracts/specs/refusal-spec.md`
- `//pattern/PATTERN-agent-name.md`
- `//PATTERN.md`
- `internal/shedbuild#engineMinimalConfig`
- `//docs/overview.md`

**Intent:** D6: the table's first seat is the lead, which carries no seat name and whose strand role is the role prefix alone; every later seat carries a name its table builder takes from the recipe or config.

In `internal/seatengine`:
- No code reads `RoleChair`, `RoleAdvisor` or `AdvisorName` any more: no seat is named `chair` and no name is formed as `advisor-<n>`. Card 20 deletes the three once no later card edits code that names them.
- The lead is the seat whose `Name` is empty, which `Table.Validate` pins to the first seat; the engine keeps calling it the chair internally (`Table.Chair` returns `Seats[0]`, or the zero seat for an empty table), and every `seat.Name == RoleChair` test becomes a test for the lead.
- `SeatRole(prefix, name)` returns the prefix alone for an empty name and `prefix-name` otherwise, so every strand role is still formed through it (PATTERN-agent-name) and reed forms the strand name once; `strandNames`, `seatSpec` and `Table.validateRoles` call it for every seat, the lead included, and `strandNames` keys the lead under "".
- `ValidateSeatName` is the seat-name rule: a non-empty name that itself passes `agentname.ValidateRole` (a letter first), so a later seat's formed role never reads as reed's numbered form of the lead's role (`agentname.MatchesRole`); D7's config check reuses it.
- `Table.Validate` (through `validateNames`) requires at least one seat; a lead with no name; every later seat a name passing `ValidateSeatName`, unique in the table; and, for a table of more than one seat, a lead stencil whose include list (`stencil.IncludeNames`, already read by `validateStencil`) holds `seat-directive-chair`. The advisors-numbered-in-order rule goes.
- `validateSeatRoles` and `validateOutputs` keep their rules, reading the lead by its empty name.
- `seatValues` fills `chair_name` with the lead's strand name and `advisor_names` with the later seats' strand names, as today, so the seat directives need no new marker.
- `doc.go`'s seat-model and table-rules sections describe the lead and the named seats.
Each refusal names its way forward (PATTERN-refusal-way-forward): a named lead ("the first seat is the lead and carries no name; drop its name"), an unnamed, invalid or duplicate later seat ("give the seat a unique name that starts with a letter"), and a multi-seat lead stencil without the chair directive ("include seat-directive-chair in the lead's stencil").
Each lands a `contracts/specs/refusal-spec.md` row beside the existing `MultiLLM` seat rows, since the `MultiLLM` row reaches them.

Callers in the same card, so the tree builds:
- `burlerengine`'s `reviewTable` and `fixTable` build their one seat with no name, so the strands read `<shortname>:<slug>:burler-review` and `<shortname>:<slug>:burler-fix`; `burlerengine/doc.go` says so.
- `burlercli`'s `Command` and `runCmd` help text and `burlercli/doc.go` name the roles `burler-review` and `burler-fix`.
- `contracts/stencils`' `roleOpeningStencils` keys the Burler leads as `burler-review` and `burler-fix`.
- `loomengine.DiscussionTable` builds its lead with no name and names each advisor `advisor-<n>` from its list position, a placeholder card 19 replaces with config names; `DiscussionWriterRole` drops its now-unused argument and returns the lead's role, `discussion`, and `newParentReviewConfig`, its one caller, calls it so.
- `shedrecipe.multiLLMSeat` refuses a `name` key on the first seat and requires it on every later seat, so the `MultiLLM` row's first seat drops the key; `multiLLMEntry` passes the index through. `internal/shedrecipe/testdata/multillm-recipe.yaml`'s first seat drops `name: chair`, its later seats keep their names, and its lead stencil `seatfixture-chair` includes `seat-directive-chair`, the fixture test seeding the shipped seat blocks with `stencilkit.SeedInto` into its stencils directory. `shedbuild`'s `engineMinimalConfig` MultiLLM entry, which `TestBuild_EveryRegisteredEngineBuilds` builds, drops the first seat's `name` and gives the lead a stencil that includes `seat-directive-chair`, seeded the same way.
- `contracts/specs/shed-recipe-spec.md`'s `MultiLLM` paragraph: the first `seats` element is the lead and carries no `name`, every later one requires a unique `name` starting with a letter.
- `PATTERN.md`'s `PATTERN-agent-name` entry line changes its last clause from "the spawning module owns its role names" to say the spawning module owns its role prefix and a later seat's suffix may come from the recipe or config, kept under the entry-line cap; `pattern/PATTERN-agent-name.md`'s background says a later seat's role suffix may come from the recipe or config, bounded by the role grammar and table uniqueness, the lead's role being the prefix alone (PATTERN-agent-name requires the background in the same commit).
- `docs/overview.md`'s seats and burler lines name the lead and the `burler-review`/`burler-fix` strands.

Tests:
- `TestTable_Validate` rows: an accepted lone lead; a named lead refused; an unnamed later seat refused; a duplicate later name refused; a numeric later name such as `2` refused; an invalid role prefix refused (the lead's formed role is the prefix, the existing row); a two-seat table whose lead stencil lacks `seat-directive-chair` refused, a lone lead with no such include accepted. The `chair`-count and advisor-order rows go.
- `TestTable_NamesAccessorsAndReservedMarkers`: `SeatRole("multi", "")` is `multi`, `SeatRole("multi", "critic")` is `multi-critic`; `Chair()` returns the first seat.
- `validTable`, `engineTable` and `role` build and address a nameless lead and named later seats (`critic`, `skeptic`); every engine, prompt, resume and smoke test above addresses the lead's strand as the prefix alone, and `TestSeatPrompts_ValuesPromptsAndSpecs` asserts a one-seat table's strand role is the prefix and a named second seat forms `prefix-name`.
- `burlerengine`'s `reviewHalfRole`/`fixHalfRole` and `burlercli`'s `reviewChairRole`/`fixChairRole` constants become `burler-review` and `burler-fix`; `chairPrompt` reads the lead's prompt under "".
- `TestDiscussionTable` asserts the lead is unnamed with role `discussion` and the advisors' strands read `discussion-advisor-<n>`; `TestNewParentReviewConfig_KeepAliveInterval`'s rows read one writer role, `discussion`, for every producer.
- `TestMultiLLMEntry_ConstructionFailures` rows: a `name` on the first seat refused, a missing or duplicate later name refused, the `AdvisorMisnamed` row gone; `validMultiLLMConfig` drops the first seat's name; `minimalEntries`' `MultiLLM` entry, `TestMultiLLMEntry_ConstructionFailures` and `TestMultiLLMEntry_ComposedTable` write a lead stencil that includes `seat-directive-chair` into `newTestEnv`'s stencils directory after seeding it with the shipped seat blocks through `stencilkit.SeedInto`.
- `withSeatTable`, `newTestEnv`, `multiLLMTable`, `TestNew_DiscussionSeatsRunsTheWriterRowOnSeats`, `envkit.FullEnv` and `TestWire_DiscussionTableEvaluatesToExpectedShape` build or read the lead unnamed; `TestSmokeBurlerReview_AttachesToALiveReviewerInsteadOfRespawning` probes role `burler-review`.

**ImpactSummary:** Stops every reader of three exported seatengine names, all listed here; strand roles of the Burler halves change from burler-*-chair to burler-*, and of a seats Discussion chair from discussion-chair to discussion.
