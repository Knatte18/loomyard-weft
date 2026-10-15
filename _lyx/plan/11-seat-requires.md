# Card 11 — Seats declare their requirements and name their model key in a refusal

**Create:**
- `plan:internal/shuttleengine#ModelKey` -> `type ModelKey struct`
- `plan:internal/shuttleengine#ModelKey.Wrap` -> `func (k ModelKey) Wrap(err error) error`

**Edit:**
- `internal/shuttleengine#Runner.checkRequirementsAtStart`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#TestRunner_Requires`
- `internal/seatengine#Seat`
- `internal/seatengine#seatSpec`
- `internal/seatengine#Engine.startSeat`
- `internal/seatengine/doc.go`
- `internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs`
- `internal/seatengine#TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair`
- `internal/seatengine#TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat`
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
D5 for the seat roles and D8's caller-wrapped refusal.

Shuttle's `Start` refusal stays a typed error wrapping `ErrRequirementsUnmet` that names the engine, each unmet requirement and the engines that meet them all, but `checkRequirementsAtStart` no longer appends its own generic way forward: the caller that resolved the role's model key adds it.
The attach refusal keeps its own way forward.
`ModelKey{Module, Key, DefaultEngine}` names where a role's model came from; `Wrap` returns any error not wrapping `ErrRequirementsUnmet` unchanged, and wraps a requirements refusal with one of three ways forward:
- a key with `Module` and `Key` set: `; way forward: set <Key> in <Module>.yaml to an alias whose engine meets them with lyx config <Module> --set <Key>=<alias>, then re-run the verb, or under the role's row re-step the row`;
- `DefaultEngine` set, for a model that named no engine and resolved no key: `; way forward: set engine: in shuttle.yaml to an engine that meets them, then re-run the verb`;
- the zero key, for a seat whose key no caller names, as a MultiLLM recipe seat: `; way forward: set the seat's model to an alias whose engine meets them, then re-run the verb, or under the role's row re-step the row`.
So no requirements refusal leaves a seat without a way forward.
`doc.go` states the split.

`Seat` gains `Requires []shuttleengine.Requirement` and `ModelKey shuttleengine.ModelKey`.
`seatSpec` sets the spec's `Requires` to the seat's list plus `RequirementPeerMessaging` when the table has more than one seat, so the chair and every advisor of a table with advisors require peer messaging (their seat directives message each other by `SendMessage`) and a lone seat adds nothing.
`startSeat` passes a start error through `seat.ModelKey.Wrap`, so a chair refusal returned by `Run` and an advisor refusal recorded in its `StartError` both name the key.
`DiscussionTable`'s lead carries `ModelKey{loom, discussion}` and, with `discussion_lenses` set, `Requires` `background-wake` (forks stay implied by `ForkSubagents`); each advisor carries `ModelKey{loom, discussion_advisors}`.
`PlanTable`'s lead carries `ModelKey{loom, plan}` and, with `plan_lenses` set, `background-wake`.
The twill directive tells the lead to fork and to end its turn while forks are outstanding, which is the rule behind both rows.
`RunOpts` gains `ReviewKey shuttleengine.ModelKey`, which `reviewTable` puts on the reviewer's seat; `burlerReviewEntry` sets it to `ModelKey{loom, fan_review}` for a row whose profile names a cluster fan, the list a fanned reviewer runs on, and leaves it zero otherwise.
The cluster reviewer's only requirement, forks, stays implied by `ForkSubagents`.
`lyx burler run` resolved no key: its `runCmd` sets `RunOpts.ReviewKey` to `ModelKey{DefaultEngine: true}`, so its cluster-fan reviewer's refusal, which already names the default engine, takes the `engine:` way forward; the fixer is a lone seat that never forks and requires nothing.
It lands no refusal-spec row, burler being out of that file's scope.
Darn, Describe and PR-Rework run lone seats and declare nothing.

`contracts/specs/refusal-spec.md`'s loom section gains rows for the Start refusal of the Discussion-Write lead (`discussion`), the Plan-Write lead (`plan`), a Discussion-Write advisor (`discussion_advisors`, recorded as the advisor's start error while the step goes on) and the cluster-fan reviewer (`fan_review`), each correctness halt with the way forward `Wrap` writes, and its shed section gains the row for a MultiLLM seat whose engine misses a requirement, with the zero key's way forward.

Tests:
- `TestRunner_Requires` asserts a refused `Start` names the engine and requirements and carries no way forward of its own, and adds `ModelKey.Wrap` rows as plain table rows that build no rig: a requirements refusal under a named key gains the way forward naming `lyx config loom --set discussion=<alias>`, under `DefaultEngine` the `engine:` one, under the zero key the seat one, and an error not wrapping `ErrRequirementsUnmet` passes unchanged under each.
- `TestSeatPrompts_ValuesPromptsAndSpecs` asserts a lone seat's spec has no `Requires`, a table of a chair and one advisor gives both specs `peer-messaging`, and a seat's own `background-wake` joins it.
- `TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat` gains a table of rows that run the engine over `RunnerShuttle` of a real `shuttleengine.Runner` on `shuttlefake.Reed` and `shuttlefake.EngineSet(&shuttlefake.Engine{})`, whose engine supports no requirement: a chair seat carrying each loom key and requirement set its builder gives it (`discussion` with `background-wake` and forks, `plan` likewise, `fan_review` with forks, and a two-seat table's chair with `peer-messaging`), each with an empty `Engine`, is refused at `Start` naming engine `fake` (the default an empty key is checked against) and the seat's key, with no `AddStrand` call on the reed.
- `TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair` gains a row whose fake shuttle starts the chair and refuses the advisor with the error a real `Runner`'s `Start` returns for a `peer-messaging` spec on `shuttlefake.Engine`: the advisor's recorded `StartError` names `discussion_advisors` and the step goes on.
- `TestDiscussionTable` and `TestPlanTable` assert each seat's `ModelKey` and that the lead requires `background-wake` exactly when lenses are set; `TestEngine_Review_TableShape` asserts the reviewer seat carries `opts.ReviewKey`; `TestBurlerReviewEntry_ClusterFan` asserts the key is `fan_review` for a cluster-fan row and zero for a solo row.
- `TestRunCmd` gains a row whose burler engine runs over `RunnerShuttle` of a real `Runner` on `shuttlefake.Reed` and `shuttlefake.EngineSet(&shuttlefake.Engine{})`, with a profile naming a cluster fan the embedded burler template defines and `--review-model` set: the envelope's error names engine `"fake"` and the `engine:` way forward, and the reed recorded no `AddStrand`, so the refusal came before any pane.- `TestDarnSpec` and `TestDescribeSpec_ComposesSpec` assert the lead seat declares no `Requires`.

**ImpactSummary:** shuttle's Start refusal drops its generic way forward; seats gain two fields used by seatSpec and startSeat, and the loom seat builders, burler's reviewer and burler run fill or wrap them.

**Commit:** 11: seats declare their requirements and name their model key in a refusal
