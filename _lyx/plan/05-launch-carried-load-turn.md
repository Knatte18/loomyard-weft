# Card 5 — launch-carried-load-turn

**Create:**
- `plan:internal/shuttleengine#Run.typeSegmentColor` -> `func (run *Run) typeSegmentColor(color segmentcolor.Color)`
- `plan:internal/shuttleengine#skillReed.AddStrand` -> `func (r *skillReed) AddStrand(spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/shuttleengine#skillReed.playLoad` -> `func (r *skillReed) playLoad(list string) error`

**Edit:**
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.loadSkillsThenPrompt` -> `func (run *Run) loadSkillsThenPrompt(promptLine string, loadOnLaunch bool, afterFirstLoad func()) (Result, error)`
- `internal/shuttleengine#Run.settleLoadTurn` -> `func (run *Run) settleLoadTurn(loader SkillLoader, skills []string, timeout time.Duration, final, send bool) (missing []string, died bool, err error)`
- `internal/shuttleengine#Run.loadSkillTurn` -> `func (run *Run) loadSkillTurn(loader SkillLoader, skills []string, timeout time.Duration, send bool) (turnEnd Event, ended, died bool, err error)`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.SendText`
- `internal/shuttleengine#skillReed.Status`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestRunner_Start_TypesTheSegmentColorAfterStartup`
- `internal/shuttleengine/doc.go`

**Intent:** Let shuttle take a skill-load turn the launch line already carries (D2, shuttle side), so it types no load message for the first turn.

`Launch` gains `LoadsSkills bool`: the launch line carries the engine's `SkillLoadMessage` for the spec's skills as its first turn, and `PromptLine` holds the prompt pointer to send after the skills.
An engine that sets it must also set `PromptLine`; an engine that defers its prompt without it keeps today's typed first load.

`Runner.start` after `awaitStartup`:
- `launch.LoadsSkills` with skills named: `loadSkillsThenPrompt(launch.PromptLine, true, color)`, where `color` calls `typeSegmentColor(strand.Color)`; the color is not typed before.
- Otherwise, as today: `typeSegmentColor(strand.Color)`, then `loadSkillsThenPrompt(launch.PromptLine, false, nil)` when `PromptLine` is set.
`typeSegmentColor` is today's color block moved out of `start` unchanged: nothing for an empty color or `spec.ColorByCaller`, and a failed play logged and never fatal.

`loadSkillsThenPrompt` with `loadOnLaunch` runs the first load turn through `settleLoadTurn(..., false, false)`, so `loadSkillTurn` sends nothing and only polls the events file from the run's offset for the first turn end, the pane's death or the timeout.
The timeout is the same one (`Spec.SkillLoadTimeout`, else `DefaultSkillLoadTimeout`), counted from the call, which is the end of `awaitStartup`.
Then `afterFirstLoad` runs (the color), then the retry for missing skills is typed as today (`send` true), then the prompt pointer through the verified send, with the prompt offset persisted past every load turn end as today.
A launch-carried turn that times out skips its skills with the timeout warning and the color and pointer follow; a pane that dies during it ends the startup died through `abandonStartup`, as today; the turn end is classified from the transcript by `ClassifySkillLoad` as today.
`afterFirstLoad` also runs after a typed first load when non-nil, and is nil on that path today.
Bound: the launch-carried load skips the verified send's landing check; its landing is proven by the turn end in the events file, and a turn that never starts or never ends is bounded by the skill-load timeout.

Test doubles: `skillReed` gains `LaunchLoad string` (the skill list a launch-carried load names; empty for none), `LaunchEndAfterPolls int` (Status calls before the launch turn ends; 0 ends it at `AddStrand`), and `TypedBeforeLaunchEnd []string` (texts sent while the launch turn had not ended).
`playLoad` is `SendText`'s scripted reaction to `LOAD:<list>` moved out unchanged (turn counting, `Hangs`, `Dies`, the `STOP:loaded <list>` line), so `SendText` and the launch path share it.
`skillReed.AddStrand` calls the embedded `AddStrand` and, with `LaunchLoad` set and `LaunchEndAfterPolls` zero, plays turn 0 through `playLoad`; `skillReed.Status` plays it on the `LaunchEndAfterPolls`-th call instead.
`SendText` records a text into `TypedBeforeLaunchEnd` while a set `LaunchLoad` has not ended.
A test turns the launch path on through `skillStartFixtureWith`'s `configure`, setting `engine.PrepareLaunch.LoadsSkills` and `reed.LaunchLoad`.

`Status` call accounting, which the rows below rely on: `awaitStartup`'s ready probe makes the first call, the color's and every verified send's liveness checks make one or more each, and `loadSkillTurn`'s poll loop makes one per iteration.
So a launch turn set to end on the fifth call ends inside `loadSkillTurn`'s polling on the correct path, while an implementation that types the color right after `awaitStartup` sends it before the fifth call and records it in `TypedBeforeLaunchEnd`.

Rows of `TestStartup_SkillLoadTurns` (new fields `onLaunch bool`, `launchEndAfterPolls int` and `color segmentcolor.Color`, the strand's color passed to `skillStartFixtureWith`):
- `a launch-carried load types no load message and the prompt follows its turn end`: `a,b` loaded, `wantTyped: "do the task"`, `wantOffset`.
- `a launch-carried turn that never ends is skipped at the timeout and the color and prompt follow`: orange, `hangs: [0]`, timeout 50ms, `wantTyped: "COLOR:orange|do the task"`, `wantSkips: 2`, `atLeastTimeout`.
- `a pane that dies during a launch-carried turn is a died startup`: `dies: [0]`, `launchEndAfterPolls: 2`, so the death plays on the first `Status` call after `awaitStartup`'s ready probe, inside `loadSkillTurn`; `wantTyped: ""`, `wantDied`, and the run record read through `soleStartupRunDir` holds `Started` true, which a death inside `awaitStartup` cannot leave.
Rows of `TestRunner_Start_TypesTheSegmentColorAfterStartup` (new fields `onLaunch bool`, `launchEndAfterPolls int`, `reports`):
- `ALaunchCarriedLoadColorsAfterItsTurnEndsThenRetriesThenPrompts`: orange, missing `b` then loaded, `wantTyped: "COLOR:orange|LOAD:b|do the task"`; this row pins the order and the typed retry, so `TestStartup_SkillLoadTurns` carries no retry row of its own.
- `ALongLaunchCarriedLoadDelaysTheColorWithoutLosingIt`: orange, `launchEndAfterPolls: 5`, `wantTyped: "COLOR:orange|do the task"`, and `TypedBeforeLaunchEnd` empty, which pins that the color waited for the turn end.
The existing rows keep the typed path and their expectations.
`doc.go`'s Segment color and Skill loading paragraphs state the two orders and the bound.

**ImpactSummary:** Adds one Launch field and re-signs three private Run methods whose only callers are in run.go on this card; engines that do not set the field keep today's typed load.
