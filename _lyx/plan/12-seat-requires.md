# Card 12 — Seats declare their requirements and name their model key in a refusal

**Edit:**
- `internal/seatengine#Seat`
- `internal/seatengine#seatSpec`
- `internal/seatengine#Engine.startSeat`
- `internal/seatengine/doc.go`
- `internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs`
- `internal/seatengine#TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair`
- `internal/seatengine#TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat`
- `internal/seatengine#fakeShuttle`
- `internal/seatengine#fakeShuttle.StartGated`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#PlanTable`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestPlanTable`
- `internal/burlerengine#RunOpts`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#TestEngine_Review_TableShape`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#TestBurlerReviewEntry_ClusterFan`
- `internal/burlercli#burlerCLI.runCmd`
- `internal/burlercli#TestRunCmd`
- `internal/darnengine#TestDarnSpec`
- `internal/landingshed#TestDescribeSpec_ComposesSpec`
- `contracts/specs/refusal-spec.md`

**Intent:**
D5 for the seat roles, wrapping their `Start` refusals with card 11's `shuttleengine.ModelKey`.

`Seat` gains `Requires []shuttleengine.Requirement` and `ModelKey shuttleengine.ModelKey`.
`seatSpec` sets the spec's `Requires` to the seat's list plus `RequirementPeerMessaging` when the table has more than one seat, so the chair and every advisor of a table with advisors require peer messaging (their seat directives message each other by `SendMessage`) and a lone seat adds nothing.
`startSeat` passes a start error through `seat.ModelKey.Wrap`, so a chair refusal returned by `Run` and an advisor refusal recorded in its `StartError` both name the key.
`DiscussionTable`'s lead carries `ModelKey{loom, discussion}` and, with `discussion_lenses` set, `Requires` `background-wake` (forks stay implied by `ForkSubagents`); each advisor carries `ModelKey{loom, discussion_advisors, Entry: <advisor name>}`, since `discussion_advisors` is a list of `{name, model}` pairs that `--set discussion_advisors=<alias>` would overwrite with a scalar.
`PlanTable`'s lead carries `ModelKey{loom, plan}` and, with `plan_lenses` set, `background-wake`.
The twill directive tells the lead to fork and to end its turn while forks are outstanding, which is the rule behind both rows.
`RunOpts` gains `ReviewKey shuttleengine.ModelKey`, which `reviewTable` puts on the reviewer's seat; `burlerReviewEntry` sets it to `ModelKey{loom, fan_review}` for a row whose profile names a cluster fan, the list a fanned reviewer runs on, and leaves it zero otherwise.
The cluster reviewer's only requirement, forks, stays implied by `ForkSubagents`.
`lyx burler run` resolved no key: its `runCmd` sets `RunOpts.ReviewKey` to `ModelKey{DefaultEngine: true}`, so its cluster-fan reviewer's refusal, which already names the default engine, takes the `engine:` way forward; the fixer is a lone seat that never forks and requires nothing.
It lands no refusal-spec row, burler being out of that file's scope.
Darn, Describe and PR-Rework run lone seats and declare nothing.

`contracts/specs/refusal-spec.md`'s loom section gains rows for the Start refusal of the Discussion-Write lead (`discussion`), the Plan-Write lead (`plan`), a Discussion-Write advisor (its entry in `discussion_advisors`, recorded as the advisor's start error while the step goes on) and the cluster-fan reviewer (`fan_review`), each correctness halt with the way forward `Wrap` writes, and its shed section gains the row for a MultiLLM seat whose engine misses a requirement, with the zero key's way forward.

Tests:
- `TestSeatPrompts_ValuesPromptsAndSpecs` asserts a lone seat's spec has no `Requires`, a table of a chair and one advisor gives both specs `peer-messaging`, and a seat's own `background-wake` joins it.
- `TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat` gains a table of rows that run the engine over `RunnerShuttle` of a real `shuttleengine.Runner` on `shuttlefake.Reed` and `shuttlefake.EngineSet(&shuttlefake.Engine{})`, whose engine supports no requirement: a chair seat carrying each loom key and requirement set its builder gives it (`discussion` with `background-wake` and forks, `plan` likewise, `fan_review` with forks), each with an empty `Engine`, is refused at `Start` naming engine `fake` (the default an empty key is checked against) and the seat's key's way forward, and a two-seat table's chair with `peer-messaging` and the zero `ModelKey`, as a MultiLLM recipe seat carries, is refused with the zero key's seat way forward; every row asserts no `AddStrand` call on the reed.
- `TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair` gains a row whose advisor start runs on a real `Runner` over `shuttlefake.Reed` and `shuttlefake.EngineSet(&shuttlefake.Engine{})` through `RunnerShuttle`, routed there by a new `fakeShuttle` field mapping a role to the `Shuttle` that starts it, while the chair stays on the fake shuttle's handle: the chair of a two-seat table requires `peer-messaging` itself, so no chair on that set could start.
  The advisor carries `ModelKey{loom, discussion_advisors, Entry: <its name>}`; its recorded `StartError` names engine `fake` and `the model of <its name> in discussion_advisors`, the reed recorded no `AddStrand`, and the step goes on.
- `TestDiscussionTable` and `TestPlanTable` assert each seat's `ModelKey` and that the lead requires `background-wake` exactly when lenses are set; `TestEngine_Review_TableShape` asserts the reviewer seat carries `opts.ReviewKey`; `TestBurlerReviewEntry_ClusterFan` asserts the key is `fan_review` for a cluster-fan row and zero for a solo row.
- `TestRunCmd` gains a row whose burler engine runs over `RunnerShuttle` of a real `Runner` on `shuttlefake.Reed` and `shuttlefake.EngineSet(&shuttlefake.Engine{})`, with a profile naming a cluster fan the embedded burler template defines and `--review-model` set: the envelope's error names engine `"fake"` and the `engine:` way forward, and the reed recorded no `AddStrand`, so the refusal came before any pane.
- `TestDarnSpec` and `TestDescribeSpec_ComposesSpec` assert the lead seat declares no `Requires`.

**ImpactSummary:** seats gain two fields used by seatSpec and startSeat, and the loom seat builders, burler's reviewer and burler run fill or wrap them.

**Commit:** 12: seats declare their requirements and name their model key in a refusal
