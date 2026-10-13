# Card 1 — The launch line carries the skill-load message

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#TestPrepare_SkillsDeferPromptPointer`
- `internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxTakesStartupSends`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#SkillLoader`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.loadSkillsThenPrompt` -> `func (run *Run) loadSkillsThenPrompt(promptLine string, color segmentcolor.Color) (Result, error)`
- `internal/shuttleengine#Run.settleLoadTurn`
- `internal/shuttleengine#Run.loadSkillTurn` -> `func (run *Run) loadSkillTurn(loader SkillLoader, skills []string, timeout time.Duration, typed bool) (turnEnd Event, ended, died bool, err error)`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.SendText`
- `internal/shuttleengine#skillReed.Status`
- `internal/shuttleengine#hookStateSkillEngine`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine#TestRunner_Start_TypesTheSegmentColorAfterStartup`
- `contracts/stencils#TestStencils_NoSkillLoadInstructions`
- `PATTERN.md`
- `docs/overview.md`

**Create:**
- `plan:internal/shuttleengine#skillFakeEngine.Prepare` -> `func (e *skillFakeEngine) Prepare(runDir string, spec Spec, cfg Config) (Launch, error)`
- `plan:internal/shuttleengine#skillReed.AddStrand` -> `func (r *skillReed) AddStrand(spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/shuttleengine#skillReed.playLoad` -> `func (r *skillReed) playLoad(list string) error`

**Intent:**
D2: a spec that names skills launches with the engine's skill-load message as the launch argument, so every launch through `start` carries its first turn and the first load turn is no longer typed.

`Claude.Prepare`: for a spec with skills, the launch argument is `c.SkillLoadMessage(spec.Skills)` in place of today's empty argument, and the prompt pointer still comes back as `Launch.PromptLine`; a fresh and a resumed launch are treated alike.
The message is checked against `maxLaunchPromptBytes` beside the pointer's check, before any artifact is written, and an over-limit message fails `Prepare` with an error naming its length, the limit and the skills.
`SkillLoadMessage` is already one line.
A spec without skills is unchanged: the pointer on the launch line and an empty `PromptLine`.
`Claude.Prepare`'s doc comment and its inline comment above the launch-argument choice, which say a spec with skills starts on an empty input box, and `claudeengine/doc.go`'s skill paragraph say the launch line carries the load message.

Shuttle side, in `Runner.start`: after `awaitStartup`, a launch with no `PromptLine` types the color as today; a launch with a `PromptLine` passes the color, empty when the strand has none or `spec.ColorByCaller` is set, to `loadSkillsThenPrompt`, which owns the color's place in the sequence.
`loadSkillsThenPrompt` runs, in order: the first load turn, which is the launch-carried one, so it is awaited and never typed; then the color through `typeColor`, display only as today, so a failed play is a Warn; then the retry turn for missing skills, typed; then the prompt pointer, through the verified send as today.
A spec with no skills types the color and then the prompt.
`settleLoadTurn` passes its `final` flag as `loadSkillTurn`'s `typed`: the first turn is the launch-carried one and the retry is the only typed one.
`loadSkillTurn` sends the load message only when `typed`, and otherwise only polls the events file from the run's offset for the first turn end, as its poll loop does today.
The timeout (`Spec.SkillLoadTimeout`, else the engine's `DefaultSkillLoadTimeout`) is measured from `loadSkillTurn`'s entry, which for the first turn is the end of `awaitStartup`.
A first turn that times out skips its skills with the `cause=timeout` warning and goes on to the color and the prompt with no retry; a pane that dies during it is a died startup through `abandonStartup`, as today, and types no color.
Bound: the launch-carried load message skips the verified send's landing check; its landing is proven instead by a turn end in the events file, and a turn that never starts or never ends is bounded by the skill-load timeout.
`Launch.PromptLine`'s and `SkillLoader`'s doc comments, and `shuttleengine/doc.go`'s skill-loading and color paragraphs, say the first load turn is carried on the launch line and the color follows it.
The orch watcher's reload through `Runner.LoadSkills` still types its load message and is unchanged.
The `claudeengine` corpus replay feeds its first line at the first poll sleep, which the awaited first turn still takes, so it needs no change.

Fakes: `skillFakeEngine.Prepare` delegates to `fakeEngine.Prepare` and, for a spec with skills, sets `Cmd` to `e.SkillLoadMessage(spec.Skills)`, as Claude does.
`skillReed.AddStrand` delegates to the embedded `fakeReed.AddStrand` and, when the spec's `Cmd` starts with `LOAD:`, plays the load reaction for it.
`playLoad` is the reaction `SendText` plays today for a typed `LOAD:` text, moved into one method both call: it counts the turn, kills the pane for a turn in `Dies`, leaves a turn in `Hangs` open, defers the turn end of a turn in `LateEnds`, and otherwise appends the turn end at once.
A launch-carried turn in `Dies` does not kill the pane at `AddStrand`: `playLoad` arms the death for the second `Status` call, the first after the startup probe's single ready tick, so the pane dies inside `loadSkillTurn`'s poll and not in the startup probe.
`skillReed` gains `LateEnds map[int]int`, a turn's number of `Status` calls before its turn end is appended, which `skillReed.Status` appends once reached, and `EndsAtColor`, the number of load turn ends appended when the first `COLOR:` text was typed, which `SendText` records.
`hookStateSkillEngine`'s doc comment drops the claim that only the hook state gets a typed first load past a busy pane: after D2 its hook fold reads idle from the launch-carried turn's end, which is what lets the color, retry and prompt sends through a pane the capture fallback reads busy.

Tests:
- `TestPrepare_SkillsDeferPromptPointer`: the skills case asserts `Launch.Cmd` holds the load message and no prompt pointer, with `PromptLine` the pointer, for a fresh and a resumed launch; the no-skills case is unchanged and pins "a run without skills launches with the prompt pointer, unchanged"; a new row with one skill name long enough that the message passes `maxLaunchPromptBytes` asserts the error and that no `prompt.md` was written.
- `TestStart_LoneEOLMarkBoxTakesStartupSends`: no turn end reaches the events file, so the first turn times out on the virtual clock; the test asserts the pane received `/color blue` and the canned prompt line, and never the load message.
- `TestStartup_SkillLoadTurns`: each row's `wantTyped` drops the first `LOAD:` entry (for example `LOAD:a,b|LOAD:b|do the task` becomes `LOAD:b|do the task`); the never-ending first turn row keeps its timeout skip, its at-least-timeout elapsed and `do the task` typed.
  The pane-dies row types nothing and also asserts that the run record reads `Started` true, so the death it reaches is `loadSkillTurn`'s after a ready startup, not the startup probe's.
  The hook-state row is renamed "the hook state's idle reading after the load turn sends the prompt into a pane the pane fallback refuses", typing `do the task`.
  These rows pin "types no load message for the first turn" and "a load turn that times out skips its skills with a warning, and a pane that dies during it ends the startup died".
- `TestRunner_Start_TypesTheSegmentColorAfterStartup`: the table gains `skills` and `reports` fields, defaulting to `a` and its loaded report; the doc comment and the first row say the color follows the skill load; every row that types a color types `COLOR:orange|do the task` and asserts `EndsAtColor` is 1, and the rows that type none assert `do the task` alone.
  A new row has the first turn end late through `LateEnds` with an orange strand and asserts `COLOR:orange|do the task` with `EndsAtColor` 1, so the color waited for the load turn's end and was not lost.
  A new row names skills `a` and `b` with a first report missing `b` and asserts `COLOR:orange|LOAD:b|do the task`, the order the record names.
  The hook-state row is renamed `TheHookStateIdleAfterTheLoadTurnColorsAPaneThePaneFallbackRefuses`.
- `TestStartup_SendFailuresTearDown`: the table gains a `reports` field; the two skill-load send rows and the prompt-offset persist row script a first report naming the skill missing, so the failing send, or `brokenRunRecordReed`'s break on a typed `LOAD:`, is the retry turn's.

`PATTERN-role-skills-typed`'s entry line keeps its id and says "lyx delivers it, on the launch line or typed" in place of "lyx types it", with the same no-stencil rule.
`docs/overview.md`'s "skills and the parent directive" bullet says the provider engine delivers a spawn's skills before the prompt, on the launch line or typed.
`TestStencils_NoSkillLoadInstructions`'s failure message, its `//testtiming:keep` reason and the header comment of its file `contracts/stencils/roleskills_test.go` say a skill is delivered by lyx in place of typed by lyx.

**ImpactSummary:** `loadSkillsThenPrompt` and `loadSkillTurn` are re-signed with one caller each (`Runner.start`, `Run.settleLoadTurn`), both on this card; `Claude.Prepare`'s output changes only for a spec with skills, whose one consumer is shuttle's `start`; the rest is test fakes and doc text in `shuttleengine`, `claudeengine`, `contracts/stencils`, `PATTERN.md` and `docs/overview.md`.

**Commit:** 1: Carry the skill-load message on the launch line
