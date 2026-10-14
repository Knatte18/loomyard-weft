# Card 7 — launch-carries-skill-load

**Create:**
- `plan:internal/shuttleengine#skillFakeEngine.Prepare` -> `func (e *skillFakeEngine) Prepare(runDir string, spec Spec, cfg Config) (Launch, error)`
- `plan:internal/shuttleengine#skillReed.AddStrand` -> `func (r *skillReed) AddStrand(spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/shuttleengine#skillReed.playLoad` -> `func (r *skillReed) playLoad(list string) error`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#TestPrepare_SkillsDeferPromptPointer`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.loadSkillsThenPrompt` -> `func (run *Run) loadSkillsThenPrompt(promptLine string, colorize func()) (Result, error)`
- `internal/shuttleengine#Run.settleLoadTurn` -> `func (run *Run) settleLoadTurn(loader SkillLoader, skills []string, timeout time.Duration, send, final bool) (missing []string, died bool, err error)`
- `internal/shuttleengine#Run.loadSkillTurn` -> `func (run *Run) loadSkillTurn(loader SkillLoader, skills []string, timeout time.Duration, send bool) (turnEnd Event, ended, died bool, err error)`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#hookStateSkillEngine`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.SendText`
- `internal/shuttleengine#skillReed.Status`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestRunner_Start_TypesTheSegmentColorAfterStartup`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine/doc.go`
- `PATTERN.md`
- `docs/overview.md`
- `contracts/stencils/roleskills_test.go`

**Uses:**
- `internal/shuttleengine/claudeengine#Claude.SkillLoadMessage`
- `internal/shuttleengine/claudeengine#maxLaunchPromptBytes`

**Intent:** D2: a spec that names skills launches with the engine's skill-load message as the launch argument, in place of today's empty input box, so every launch through `start` carries its first turn, one typed send goes away, and card 6's events proof is available on every `start`.

Engine (`claudeengine`): in `Prepare`, a spec with skills sets the launch argument to `c.SkillLoadMessage(spec.Skills)` and keeps returning the prompt pointer as `Launch.PromptLine`; a spec without skills keeps the pointer on the launch line and an empty `PromptLine`, fresh or resumed, unchanged.
The message is one line by construction; `Prepare` refuses, before writing any artifact, a message longer than `maxLaunchPromptBytes`, naming its length, the limit and the skill count, and its way forward, to name fewer skills.
The launch argument and the skill names stay under `claudeengine`, per `PATTERN-shuttle-provider-seam`.
Update `Prepare`'s doc comment and the `buildLaunchCmd` doc sentence that says a launch loading skills sends no first message; `buildLaunchCmd`'s signature is unchanged.
The `Launch.PromptLine` field doc in `internal/shuttleengine/engine.go` (`Launch`, a struct field, so no arrow) says that a set `PromptLine` comes with a `Cmd` that carries the engine's skill-load message for the spec's skills as the first turn.

Shuttle (`run.go`):
- `start` types the color right after `awaitStartup` only for a spec without skills, as today; for a spec with skills it passes a `colorize` func (today's color block: skip without a color or with `ColorByCaller`, Warn on a failed play) to `loadSkillsThenPrompt`, which runs it after the first load turn ends.
- `loadSkillsThenPrompt`, for a spec with skills: settles the first load turn with `send` false (nothing typed; it awaits the launch-carried turn), then calls `colorize`, then settles the retry turn for the missing skills with `send` true, then persists the prompt offset and delivers the prompt pointer through the verified send, each typed step through the verified send as today.
  A spec without skills goes straight to the prompt offset and the prompt, as today.
  A died pane or a load error during the first turn ends the start through `abandonStartup` before the color, as today's died and error paths do.
- `settleLoadTurn` passes `send` to `loadSkillTurn`; `loadSkillTurn` skips `sendVerified` when `send` is false and otherwise is unchanged.
  The first turn's deadline is `Spec.SkillLoadTimeout`, else the engine's `DefaultSkillLoadTimeout`, measured from the call, which is the end of `awaitStartup`; a launch-carried turn that never starts or never ends skips its skills with a warning at that deadline, and the color and prompt pointer follow.
  The first turn is classified from the transcript through `ClassifySkillLoad` as today, its timeout and a pane death during it ending as today's do.
- Update the doc comments of the three re-signed methods and `start` for the order: load turn end, color, retry turn, prompt pointer.
Bound, and what this lets through: the launch-carried message skips the verified send's landing check; its landing is proven instead by a turn end in the events file, and a turn end that never comes is bounded by the skill-load timeout.

The `Spec.Segment` field doc in `internal/shuttleengine/spec.go` (a struct field, so no arrow), which says shuttle types the color command after startup, says the color follows the first load turn for a spec with skills and startup otherwise.

Fakes (`fakes_test.go`):
- `hookStateSkillEngine`'s doc comment, which says a busy pane passes once a typed first load ends at once, says instead that the launch-carried load turn's end, read from hook state, is what lets a busy-looking pane through, and that a capture holding a typed `LOAD:` is now a retry turn; `IdleSession`'s logic is unchanged.
- `skillFakeEngine.Prepare` calls the embedded `fakeEngine.Prepare` and, for a spec naming skills, sets the returned `Cmd` to `e.SkillLoadMessage(spec.Skills)`, mirroring the engine contract; `startSendEngine` and `hookStateSkillEngine` inherit it through their embedded `*skillFakeEngine`.
- `skillReed` gains `AddStrand`, recording the added `Cmd` before delegating, and `playLoad`, today's scripted reaction to a typed `LOAD:<list>` moved out of `SendText` (count the turn; a turn in `Dies` kills the pane, one in `Hangs` never ends, one in the new `LateTurns` ends later, any other appends `STOP:loaded <list>` to the events file).
  `SendText` calls `playLoad` for a typed `LOAD:` and, before that, records how many turn ends the events file holds in a new field `EndsAtSend []int`, one entry per typed text.
  `CapturePane`, on its first call when the recorded `Cmd` starts with `LOAD:`, plays the launch-carried turn through `playLoad` before building the capture, outside the `fakeReed` lock; a first capture is the moment the provider is up and its launch turn begins.
  New field `LateTurns map[int]int`: a turn in it ends, appending its `STOP:` line, on the N-th `Status` call after it began; `Status` counts the calls and appends a due turn end.

Tests:
- `TestStartup_SkillLoadTurns`: every row's `wantTyped` drops the first, no longer typed, load message: "every skill loads in one turn before the prompt", "a line appended after the last load turn end lies before the prompt offset", "unknown skill is skipped at once with no retry", "unverified first turn confirms with no retry" and "first turn that never ends is skipped at the timeout with no retry" type `do the task`; "missing skill is retried once, naming only it", "skill still missing after the retry is skipped" and "unverified retry confirms with no further turn" type `LOAD:b|do the task`; "pane that dies during a load turn is a died startup" types nothing and still wants the died start with no pane tail; the `hookState` row, renamed "a session start and the launch turn's end let the prompt into a pane the pane fallback refuses", types `do the task`.
- `TestRunner_Start_TypesTheSegmentColorAfterStartup`: rename the first row "ColoredStrandIsColoredAfterTheLoadTurn" with `wantTyped` `COLOR:orange|do the task`, and every other existing row drops `LOAD:a|`.
  A row field `reports` overrides the shared `loaded` reports, `lateTurns` sets `skillReed.LateTurns`, `noSkills` drops the spec's skills, and `wantEndsAtColor` asserts the `EndsAtSend` entry of the `COLOR:` send.
  New rows: "TheColorPrecedesTheRetryTurn" (first report `Missing: [a]`, then loaded) types `COLOR:orange|LOAD:a|do the task`, the color, retry and prompt order the criterion names; "ALongLoadTurnDelaysTheColorWithoutLosingIt" (`lateTurns` `{0: 3}`) types `COLOR:orange|do the task` with `wantEndsAtColor` 1, which fails if the color went out before the launch turn ended or not at all; "ASpecWithoutSkillsIsColoredRightAfterStartup" (`noSkills`) types `COLOR:orange|do the task` with `wantEndsAtColor` 0.
- `TestStartup_SendFailuresTearDown`: the two skill rows whose failure was the typed first load, "a skill-load send that stays busy" and "a skill load held past the send window by an active operator", and "a prompt-offset persist that fails", whose `brokenRunRecordReed` breaks the record on a typed load, gain a row field `reports` scripting a first turn with `Missing: [a]` and then a loaded retry, so the retry turn is the typed load each row fails; rows without the field keep today's single loaded report.
- `TestPrepare_SkillsDeferPromptPointer`: for a spec with skills, assert `Launch.Cmd` contains `shell.ForGOOS().Quote(New().SkillLoadMessage(skills))` and no `Read ` pointer, and `PromptLine` is the pointer; the no-skills half is unchanged, pinning that a run without skills launches with the prompt pointer; add a case whose skill list makes the message longer than `maxLaunchPromptBytes`, which `Prepare` refuses with no artifact written.
No package gains a top-level test.

`PATTERN-role-skills-typed` changes with D2, in this commit: reword its `PATTERN.md` entry line to exactly the line below, which is shorter than today's, so `lyx loom lint-pattern` passes with the overview at its headroom line:

```
- `PATTERN-role-skills-typed` — Loading a spawned session's skill: the spawner names it on the launch spec, lyx delivers it on the launch line or typed; no stencil asks for one. (test)
```

`contracts/stencils/roleskills_test.go`'s file comment says lyx delivers the skill, on the launch line or typed, in place of "lyx types it", and `TestStencils_NoSkillLoadInstructions`' error message says "delivered by lyx" in place of "typed by lyx".
`docs/overview.md`'s skills line says a spawn's skills are named on the launch spec and delivered by the provider engine before the prompt, on the launch line or typed.

Docs: `internal/shuttleengine/doc.go`'s skill-loading paragraph says the engine puts its load message on the launch line and returns the pointer as `PromptLine`, shuttle awaits that first turn's end without typing it, then types the color, the one retry turn and the prompt pointer, and the segment-color paragraph says the color follows the first load turn for a spec with skills and startup otherwise.
`internal/shuttleengine/claudeengine/doc.go`'s skill paragraph says a spec that names skills launches with `SkillLoadMessage` as its first message, in place of an empty input box.

**ImpactSummary:** claudeengine's launch argument for a skills spec, shuttle's start sequence for skills (first load turn awaited not typed, color after it) through three re-signed private methods, the skill test fakes and three startup tests; PATTERN entry, overview line, roleskills test wording and two doc.go files follow.

**Commit:** 7: Carry the skill-load message on the launch line and color after the load turn
