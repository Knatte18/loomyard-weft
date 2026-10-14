# Card 7 — launch-line-skill-load

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#buildLaunchCmd`
- `internal/shuttleengine/claudeengine#TestPrepare_SkillsDeferPromptPointer`
- `internal/shuttleengine/claudeengine#TestPrepare_RejectedBeforeArtifacts`
- `internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxTakesStartupSends`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#SkillLoader`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.loadSkillsThenPrompt` -> `func (run *Run) loadSkillsThenPrompt(promptLine string, color segmentcolor.Color) (Result, error)`
- `internal/shuttleengine#Run.settleLoadTurn` -> `func (run *Run) settleLoadTurn(loader SkillLoader, skills []string, timeout time.Duration, final, typed bool) (missing []string, died bool, err error)`
- `internal/shuttleengine#Run.loadSkillTurn` -> `func (run *Run) loadSkillTurn(loader SkillLoader, skills []string, timeout time.Duration, typed bool) (turnEnd Event, ended, died bool, err error)`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.SendText`
- `internal/shuttleengine#skillReed.Status`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine#skillStartFixtureWith`
- `internal/shuttleengine#hookStateSkillEngine`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestRunner_Start_TypesTheSegmentColorAfterStartup`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine/doc.go`
- `PATTERN.md`
- `docs/overview.md`
- `contracts/stencils/roleskills_test.go`

**Create:**
- `plan:internal/shuttleengine#Run.typeSegmentColor` -> `func (run *Run) typeSegmentColor(color segmentcolor.Color)`
- `plan:internal/shuttleengine#skillReed.AddStrand` -> `func (r *skillReed) AddStrand(spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/shuttleengine#skillReed.playLoad` -> `func (r *skillReed) playLoad(list string) error`

**Intent:**
D2: a spec that names skills launches with the engine's skill-load message as the launch argument, and shuttle types no first load turn.

Engine side.
`Claude.Prepare` puts `c.SkillLoadMessage(spec.Skills)` on the launch line where it puts an empty argument today, and still returns the prompt pointer as `Launch.PromptLine`.
Before writing any artifact it refuses a load message that holds a newline or carriage return, or that is longer than `maxLaunchPromptBytes`, with an error naming the one-line rule or the launch limit, beside the pointer's own limit check; the resume line is unchanged.
`buildLaunchCmd`'s doc comment says its first-message argument is the prompt pointer or, for a spec naming skills, the skill-load message.
`claudeengine/doc.go`'s skill-loading paragraph says the launch line carries the load message.
The contract shuttle relies on is stated on `SkillLoader`'s doc comment: an engine's `Prepare` puts `SkillLoadMessage(spec.Skills)` on the launch line of a spec that names skills, so its first load turn is never typed; `Launch.PromptLine`'s comment says the pointer then follows the load turns.

Shuttle side.
`typeSegmentColor` plays the segment color through `typeColor` and the verified send and logs a failed play, the code `Runner.start` holds inline today; a zero color types nothing.
`Runner.start` resolves the color once (the strand's color, none when `spec.ColorByCaller`).
A spec without skills types the color right after `awaitStartup`, as today, then calls `loadSkillsThenPrompt` when `launch.PromptLine` is set.
A spec with skills calls `loadSkillsThenPrompt(launch.PromptLine, color)` and types no color itself.
`loadSkillsThenPrompt`, for a spec with skills, settles the first load turn with `typed` false: `loadSkillTurn` sends nothing and polls the events file from the run's offset for the first turn end, its deadline the load timeout (`Spec.SkillLoadTimeout`, else the engine's `DefaultSkillLoadTimeout`) measured from that call, which follows the end of `awaitStartup`.
A turn end is classified from the transcript as today, a timeout skips the skills with a warning, and a pane that dies ends the start died, as today.
Then it types the color, then the retry turn for missing skills with `typed` true, then the prompt pointer, each through the verified send; for a spec without skills it sends only the prompt pointer, as today.
The prompt offset is still the events file's size just before the pointer is sent, past every load turn end.
Bound: the launch-carried load message skips the verified send's landing check; its landing is proven by its turn end in the events file, and a turn that never starts or never ends is bounded by the load timeout, after which the skills are skipped with a warning and the color and pointer follow.
The retry, the color and the pointer still go through the verified send, per `PATTERN-pane-text-injection`.

Fixtures.
`skillReed` gains the launch-carried turn: its new `AddStrand` records the load list of a launch `Cmd` that starts with `LOAD:`, and `CapturePane` plays that turn on its first call through `playLoad` (the reaction `SendText` holds today for a typed `LOAD:`, moved into one method: a hung turn writes nothing, a dying turn marks the pane dead, any other appends `STOP:loaded <list>`), and from then on shows the launch's load text in every capture as if typed.
Playing on the first capture keeps a dying load turn a death after startup was ready, not during it.
A new field `Late map[int]int` makes a load turn end only at that many `Status` calls after it starts, for a long-running load turn, and `SendText` records, for each `COLOR:` typed, whether the events file already holds a turn end.
`skillStartFixtureWith` sets `PrepareLaunch.Cmd` to `LOAD:` plus the comma-joined skills when the spec names skills, and leaves `PromptLine` set either way, so a spec without skills takes the no-skills branch with a pointer to send.
`hookStateSkillEngine`'s doc comment says a capture is idle when it holds a launch-carried or typed skill load.

Tests.
`TestStartup_SkillLoadTurns`: every `wantTyped` drops its leading launch-carried `LOAD:<all>|` (for example `LOAD:a,b|LOAD:b|do the task` becomes `LOAD:b|do the task`, and the died row types nothing), the timeout row keeps its at-least-the-timeout virtual elapsed, and the hook-state row is renamed `a session start alone lets the sends into a pane the pane fallback refuses`.
`TestRunner_Start_TypesTheSegmentColorAfterStartup` gains per-row `skills`, `reports`, `hangs`, `late` and `skillLoadTimeout` fields, defaulting to today's skills `a`, the `loaded` report and none, and keeps the fixture's elapsed return for the row that asserts it; its rows want `COLOR:orange|do the task` in place of `COLOR:orange|LOAD:a|do the task` and `do the task` in place of `LOAD:a|do the task`, the first renamed `ColoredStrandIsColoredAfterTheLaunchLoadTurn`; new rows:
- `ColorThenRetryThenPromptInOrder`: reports `Missing: [b]` then `Loaded: [b]` over skills `a,b`, wanting `COLOR:orange|LOAD:b|do the task`;
- `ALongLoadTurnDelaysTheColorWithoutLosingIt`: `Late` holds turn 0 for three status calls, wanting `COLOR:orange|do the task`, the color typed only after the turn end was in the events file, and a virtual elapsed above zero;
- `ALoadTurnThatTimesOutStillGetsTheColorAndPrompt`: turn 0 hangs under a short `SkillLoadTimeout`, wanting `COLOR:orange|do the task`; the skip warning itself stays pinned by `TestStartup_SkillLoadTurns`'s timeout row;
- `ColoredStrandWithoutSkillsIsColoredBeforeThePrompt`: no skills, wanting `COLOR:orange|do the task`, which reaches `Runner.start`'s no-skills branch.
`TestStartup_SendFailuresTearDown`: a skill row's launch `Cmd` becomes `LOAD:a` and its reports name `a` missing then loaded, so the typed send that stays busy, is held by an active operator or breaks the record is the retry `LOAD:a`; a new per-row `reports` field carries this, and the rows' wanted sentinels are unchanged.
`TestPrepare_SkillsDeferPromptPointer` asserts a skills launch line carries the quoted load message and no pointer, with the pointer as `PromptLine`.
`TestPrepare_RejectedBeforeArtifacts` gains the rows `skill_load_over_launch_limit` (one skill name of `maxLaunchPromptBytes` bytes, wanting `launch limit`) and `skill_load_not_one_line` (a skill name holding a newline, wanting `one line`).
`TestStart_LoneEOLMarkBoxTakesStartupSends` asserts the pane received `/color blue` and the prompt line `Read the prompt.`, and never the load message, since the canned launch carries it; its load turn ends at the timeout on the virtual clock.
`claudeengine`'s corpus replay needs no change: its skill case's load turn end is appended at the replay's first sleep whether or not the load was typed, and this card's package gate runs it.

Docs, in the same commit.
`internal/shuttleengine/doc.go`'s segment-color and skill-loading paragraphs say the load message rides the launch line, the color follows the first load turn's end for a spec with skills, and the retry and pointer follow it.
`PATTERN.md`'s `PATTERN-role-skills-typed` line becomes the line below; it is one byte longer than today's, within `PATTERN.md`'s four bytes of headroom, so `lyx loom lint-pattern` passes:

```markdown
- `PATTERN-role-skills-typed` — Giving a spawned session a skill: its spawner names it on the launch spec and lyx delivers it, on the launch line or typed; no stencil asks for one. (test)
```

`docs/overview.md`'s skills line says a spawn's skills are delivered by the provider engine, on the launch line or typed, before the prompt.
`contracts/stencils/roleskills_test.go`'s header comment says lyx delivers the skill instead of typing it.

**ImpactSummary:** Every shuttle start whose spec names skills changes its first turn and send order; the three re-signed `run.go` methods have no callers outside `run.go`, and Claude's launch line changes only for a spec naming skills.
