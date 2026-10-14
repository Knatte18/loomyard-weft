# Card 7 — The launch line carries the skill-load message

**Create:**
- `plan:internal/shuttleengine#skillReed.AddStrand` -> `func (r *skillReed) AddStrand(spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/shuttleengine#skillReed.playLoadTurn` -> `func (r *skillReed) playLoadTurn(list string) error`

**Edit:**
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.loadSkillsThenPrompt` -> `func (run *Run) loadSkillsThenPrompt(promptLine string, color segmentcolor.Color) (Result, error)`
- `internal/shuttleengine#Run.settleLoadTurn` -> `func (run *Run) settleLoadTurn(loader SkillLoader, skills []string, timeout time.Duration, final, launched bool) (missing []string, died bool, err error)`
- `internal/shuttleengine#Run.loadSkillTurn` -> `func (run *Run) loadSkillTurn(loader SkillLoader, skills []string, timeout time.Duration, launched bool) (turnEnd Event, ended, died bool, err error)`
- `internal/shuttleengine#SkillLoader`
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.SendText`
- `internal/shuttleengine#skillReed.Status`
- `internal/shuttleengine#hookStateSkillEngine`
- `internal/shuttleengine#skillStartFixtureWith`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine#TestRunner_Start_TypesTheSegmentColorAfterStartup`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#maxLaunchPromptBytes`
- `internal/shuttleengine/claudeengine#launchPointer`
- `internal/shuttleengine/claudeengine_test#replayCorpusCase`
- `internal/shuttleengine/claudeengine#TestPrepare_SkillsDeferPromptPointer`
- `internal/shuttleengine/claudeengine#TestPrepare_PromptLaunchLimit`
- `internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxTakesStartupSends`
- `internal/shuttleengine/claudeengine/doc.go`
- `contracts/stencils/roleskills_test.go`
- `PATTERN.md`
- `docs/overview.md`

**Intent:** D2, under `PATTERN-shuttle-provider-seam`, `PATTERN-pane-text-injection`, `PATTERN-guarded-interrupt` and a reworded `PATTERN-role-skills-typed`.

Engine side (`internal/shuttleengine/claudeengine/claudeengine.go`): for a spec that names skills, `Prepare` passes `c.SkillLoadMessage(spec.Skills)` as the launch argument in place of today's empty one, and still returns the prompt pointer as `Launch.PromptLine`; a spec without skills keeps the pointer on the launch line, unchanged.
Before any artifact is written, it refuses a skill-load message that holds a newline or exceeds `maxLaunchPromptBytes`, naming the launch limit like the pointer check.
The message text stays in `skillload.go`, so no Claude wording enters `shuttleengine`.
The doc comments of `maxLaunchPromptBytes` and `launchPointer` (`command.go`) stop saying the argument is always the pointer: the launch argument is the pointer, or for a spec that names skills the skill-load message, and either is held to the bound.

Contract (`internal/shuttleengine/engine.go`): the `SkillLoader` doc says an engine with the capability launches a spec naming skills with `SkillLoadMessage(spec.Skills)` as its launch argument, so that load turn is the session's first turn; the `Launch.PromptLine` field comment says the pointer then follows the skill loads.

Shuttle side (`internal/shuttleengine/run.go`):
- `start`, after `awaitStartup`: for a spec with skills it calls `loadSkillsThenPrompt(launch.PromptLine, color)`, where `color` is `strand.Color`, empty when `spec.ColorByCaller`; for a spec without skills it types the color as today and then, when `PromptLine` is set, calls `loadSkillsThenPrompt(launch.PromptLine, "")`.
- `loadSkillsThenPrompt` runs the first load turn with `launched` true, then types the color through `typeColor` (display only; a failed play is a Warn, as today), then the retry turn for missing skills with `launched` false, then the prompt pointer, each typed send through `sendVerified`.
- `settleLoadTurn` passes `launched` through; `loadSkillTurn` with `launched` true types nothing and only waits for the first turn end past the run's offset, polling as today.
  Its deadline is the skill-load timeout (`Spec.SkillLoadTimeout`, else `DefaultSkillLoadTimeout`) measured from its call, which is the end of `awaitStartup`, so a launch-carried turn that never starts or never ends skips its skills with a warning and the color and prompt follow; a pane that dies during it is a died startup, as today.
  The turn is classified from the transcript by `ClassifySkillLoad` as today.
Bound: the launch-carried load skips the verified send's landing check; its landing is proven by the turn end in the events file instead, bounded by the skill-load timeout. Every send that stays typed still goes through `sendVerified`.

Fakes (`internal/shuttleengine/fakes_test.go`):
- `skillReed` gains `LaunchCarriesLoad bool`, `FirstTurnAfter time.Duration` and `Now func() time.Time`.
- `playLoadTurn` holds the scripted reaction `SendText` plays today for a typed `LOAD:<list>` (count the turn, die, hang, or append `STOP:loaded <list>`); `SendText` calls it.
- `AddStrand` calls the embedded `fakeReed.AddStrand`, records `Now()` as the launch time and, when `LaunchCarriesLoad` holds and `FirstTurnAfter` is zero, plays turn 0 through `playLoadTurn("launch")` at once.
- `Status`, when `LaunchCarriesLoad` holds, `FirstTurnAfter` is positive and turn 0 has not played, plays it on the first call at which `Now()` is at least `FirstTurnAfter` past the launch time, so the first turn ends after that much virtual time whatever number of `Status` calls `awaitStartup` and the load wait made; with `fastConfig`'s probe ready on its first tick, that call is inside `loadSkillTurn`'s wait.
- `skillStartFixtureWith` sets `LaunchCarriesLoad` from `len(spec.Skills) > 0` and `Now` to its fake clock's `Now` before `configure` runs.
- `hookStateSkillEngine`'s doc comment stops saying the pane fallback refuses the pane "until the first load": the first load rides the launch line, so a capture holding a typed `LOAD:` is now the retry's, and the hook state is what lets the color and prompt sends past a busy-looking pane.

Tests:
- `TestStartup_SkillLoadTurns`: every row's `wantTyped` drops the first `LOAD:...`, since turn 0 now rides the launch line (for example `do the task`, `LOAD:b|do the task` for the retry rows); `first turn that never ends` still scripts `hangs` on turn 0 and wants the timeout skip and elapsed at least the timeout.
  `pane that dies during a load turn` sets `FirstTurnAfter` to one poll interval through `configure`, so `awaitStartup` reports ready over a live pane first and turn 0's death lands inside `loadSkillTurn`'s launch-carried wait; it wants nothing typed and a died startup, and fails if that wait's died check is broken.
  The `hookState` row keeps pinning that the typed prompt goes out over a busy-looking pane once the hook state is idle.
- `TestRunner_Start_TypesTheSegmentColorAfterStartup`: the table gains per-row `skills`, `reports` and `firstTurnAfter` fields (defaulting to skills `a` and the `loaded` report), and `wantMinElapsed time.Duration`, read from `skillStartFixtureWith`'s elapsed result.
  Rows want `COLOR:orange|do the task` in place of `COLOR:orange|LOAD:a|do the task`; new rows `ColorRetryAndPromptFollowTheLoadTurnInOrder` (skills `a, b`, reports `Missing: [b]` then loaded; want `COLOR:orange|LOAD:b|do the task`) and `ALongLaunchCarriedLoadTurnDelaysTheColorWithoutLosingIt` (`firstTurnAfter` three poll intervals of `fastConfig`; want `COLOR:orange|do the task`, a started run, and `wantMinElapsed` three poll intervals, so the color was typed only after the turn end and still typed).
- `TestStartup_SendFailuresTearDown`: the table gains a per-row `reports` field in place of the one shared `Reports` value, its `skillReed` sets `LaunchCarriesLoad` for the skill rows, and the three skill rows report `Missing: ["a"]` then loaded, so the typed retry `LOAD:a` is the send that stays busy, is held by an active operator, or (through `brokenRunRecordReed.SendText`, unchanged, which still fires on a typed `LOAD:`) breaks the record before the prompt-offset persist; rename the first two `a retry skill-load send that stays busy` and `a retry skill load held past the send window by an active operator`.
- `TestPrepare_SkillsDeferPromptPointer` (`internal/shuttleengine/claudeengine/prepare_test.go`): with skills, `Launch.Cmd` carries `SkillLoadMessage(skills)` quoted as the launch argument (asserted on non-Windows, where the posix quoting keeps the text verbatim) and no pointer, and `PromptLine` is the pointer; without skills it is unchanged — the "run without skills launches with the prompt pointer" criterion.
- `TestPrepare_PromptLaunchLimit`: new subtests refuse a skill list whose load message exceeds `maxLaunchPromptBytes` and a skill name holding a newline, each naming the launch limit or the newline, with no artifact written.
- `TestStart_LoneEOLMarkBoxTakesStartupSends` (`start_test.go`): the load message is no longer pasted; it asserts `/color blue` and `Read the prompt.` were pasted (the canned launch's load turn never ends in this fake, so its skills are skipped at the virtual timeout first), and that the load message was not pasted.
- `replayCorpusCase` (`internal/shuttleengine/claudeengine/corpus_test.go`), whose skilled cases (`skill-load-turn-end`, `hook-capture`) start over `shuttlefake.Engine`: its first line, the load turn end Start consumes, is now read by the launch-carried wait rather than after a typed load, and its replay clock appends that line on the first `Sleep`, which `loadSkillTurn`'s poll still takes; the card checks both cases still consume exactly that line and adjusts the replay's `consumed` handling only if they do not.
No new top-level test.

Docs in the same commit: `internal/shuttleengine/doc.go` skill-loading paragraph and the color sentence (the color follows the launch-carried load turn for a spec with skills); `internal/shuttleengine/claudeengine/doc.go` (a spec that names skills launches with the load message as its argument, the pointer returned as `Launch.PromptLine`); `PATTERN.md`'s `PATTERN-role-skills-typed` entry line becomes "the spawning module names it on the launch spec and lyx delivers it, on the launch line or typed; no stencil asks an agent to load a skill.", keeping the id and the `(test)` mark and staying under the entry-line cap; `docs/overview.md`'s skills line ("typed by the provider engine" becomes "delivered by the provider engine, on the launch line or typed"); the header comment of `contracts/stencils/roleskills_test.go` and its violation message say "delivered by lyx" in place of "types it"/"typed by lyx".

**ImpactSummary:** Three unexported run.go methods re-signed, each with its one caller on this card; one engine's Prepare changes its launch argument for skilled specs only; every engine's first load turn stops being typed, which reaches the shuttleengine skill tests, claudeengine's start test and corpus replay listed here, and no other caller asserts a typed first load.

**Commit:** 7: shuttle: carry the skill-load turn on the launch line
