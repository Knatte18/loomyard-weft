# Card 6 — launch-carried-load-turn

**Create:**
- `plan:internal/shuttleengine#Run.typeSegmentColor` -> `func (run *Run) typeSegmentColor(color segmentcolor.Color)`
- `plan:internal/shuttleengine#skillReed.playLoad` -> `func (r *skillReed) playLoad(list string) error`

**Edit:**
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.loadSkillsThenPrompt` -> `func (run *Run) loadSkillsThenPrompt(promptLine string, launchCarried bool, color segmentcolor.Color) (Result, error)`
- `internal/shuttleengine#Run.settleLoadTurn` -> `func (run *Run) settleLoadTurn(loader SkillLoader, skills []string, timeout time.Duration, final, typed bool) (missing []string, died bool, err error)`
- `internal/shuttleengine#Run.loadSkillTurn` -> `func (run *Run) loadSkillTurn(loader SkillLoader, skills []string, timeout time.Duration, typed bool) (turnEnd Event, ended, died bool, err error)`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.SendText`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine#skillReed.Status`
- `internal/shuttleengine#skillStartFixtureWith`
- `internal/shuttleengine#hookStateSkillEngine`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestRunner_Start_TypesTheSegmentColorAfterStartup`
- `internal/shuttleengine/doc.go`

**Intent:** D2, shuttle's side: a launch whose line already carries the skill-load message has its first load turn waited for, not typed.

`Launch` gains the field `SkillLoadOnLaunch bool`, documented as: the launch line carries the engine's `SkillLoadMessage` for the spec's skills as its first turn, and `PromptLine` holds the prompt pointer for shuttle to send after the skills; an engine sets it only beside a non-empty `PromptLine`.

`Runner.start`, after `awaitStartup`, computes `launchCarried := launch.SkillLoadOnLaunch && len(spec.Skills) > 0` and the color to type (the strand's color, empty when `spec.ColorByCaller`).
When not launch-carried it types the color right after `awaitStartup` as today, through `typeSegmentColor`, which holds today's best-effort `typeColor` call and its `Warn` and types nothing for an empty color.
When `launch.PromptLine` is set it calls `loadSkillsThenPrompt(launch.PromptLine, launchCarried, color)`.

`loadSkillsThenPrompt`, for a launch-carried first turn, settles that turn untyped (`settleLoadTurn(..., false, false)`), then types the color through `typeSegmentColor`, then runs the retry turn for the missing skills typed, then sends the prompt pointer, each send through the verified send (`PATTERN-pane-text-injection`).
Otherwise it runs today's typed first turn and retry (color already typed by `start`).
A died first turn and a load error still end through `abandonStartup` as today, and the prompt offset is still taken just before the prompt send.
`settleLoadTurn` passes `typed` to `loadSkillTurn`, which sends the load message only when `typed` is true; untyped it only polls the events file for the first turn end past the run's offset, the pane's death or the timeout.
The timeout is `Spec.SkillLoadTimeout`, else the engine's `DefaultSkillLoadTimeout`, and for the untyped turn it runs from the end of `awaitStartup`, since nothing is typed between.
Bound: the launch-carried message skips the verified send's landing check; its landing is proven by the turn end in the events file, and a turn that never starts or never ends is bounded by that timeout, after which its skills are skipped with a warning and the color and prompt pointer follow.
An engine that leaves `SkillLoadOnLaunch` false keeps today's typed first turn.

Test doubles (`fakes_test.go`, `startup_test.go`):
- `skillReed` gains `LaunchLoad string` (the skill list a launch line carried) and `DelayLaunchEnd int`.
  `playLoad` holds `SendText`'s scripted reaction to a load (turn counting, `Hangs`, `Dies`, the `STOP:loaded <list>` append), and `SendText` calls it for a typed `LOAD:` text.
  `CapturePane` plays `LaunchLoad` once, at its first call, as load turn 0, so the pane is live when the startup probe first reads it and dies, hangs or ends only afterwards; from then on its capture shows a `LOAD:<list>` line ahead of the sent texts, as the pane shows the launch prompt (which keeps `hookStateSkillEngine`'s idle reading).
  With `DelayLaunchEnd` n > 0 the launch turn's end is appended only on the n-th `Status` call after the play, which `Status` counts.
  `SendText` also records, for each `COLOR:` text, whether the launch turn's `STOP:loaded` line was already in the events file at that send, in a `ColorAfterLaunchEnd []bool` field, so a test can tell a color typed after the load turn ended from one typed before it.
- `hookStateSkillEngine`'s doc comment says a capture is idle when it holds a `LOAD:` line, the launch-carried load the skill reed shows or a typed retry, in place of "a typed skill load".
- `skillStartFixtureWith` sets `PrepareLaunch.SkillLoadOnLaunch` and `reed.LaunchLoad` (the skills joined by commas) when the spec names skills, before `configure` runs, so a test can clear both to drive the typed path.

Tests:
- `TestStartup_SkillLoadTurns` rows type no first load: `every skill loads…`, `a line appended…`, `unknown skill…`, `unverified first turn…` and the hook-state row expect `do the task`, the hook-state row renamed `a session start alone sends the prompt into a pane the pane fallback refuses`; the missing, still-missing and unverified-retry rows expect `LOAD:b|do the task`; the never-ending first turn expects `do the task` with the elapsed at least the timeout and two `skill skipped` warnings (the timeout criterion); the pane-dies row expects nothing typed and a died start (the died criterion).
  New row `a launch that does not carry the load types it`: configure clears `SkillLoadOnLaunch` and `LaunchLoad`, and expects `LOAD:a,b|do the task`.
- `TestRunner_Start_TypesTheSegmentColorAfterStartup` rows expect the color after the load turn: `COLOR:orange|do the task` for the colored rows and `do the task` for the uncolored ones.
  Every launch-carried colored row asserts `ColorAfterLaunchEnd` is `[true]`, so a color typed right after `awaitStartup`, before the load turn ends, fails it; with `DelayLaunchEnd: 3` that assertion is what pins the delay.
  New rows: `NoSkillsColorsRightAfterStartup` (no skills, so no launch-carried turn) expects `COLOR:orange|do the task`; `ColorPrecedesTheRetryTurn` (first report missing `b`) expects `COLOR:orange|LOAD:b|do the task`; `ALongLoadTurnDelaysTheColorWithoutLosingIt` (`DelayLaunchEnd: 3`) expects `COLOR:orange|do the task` and a virtual elapsed above zero, read from the fixture's returned duration; `ColorFollowsATimedOutLoadTurn` (`hangs` turn 0, `SkillLoadTimeout: 50 * time.Millisecond`) expects `COLOR:orange|do the task`.
  Its doc comment says the color follows the launch-carried load turn and precedes the retry.
The typed-path tests (`TestStartup_SendFailuresTearDown`, whose engine leaves the flag false) stay as they are.

Docs: `internal/shuttleengine/doc.go`'s skill-loading paragraph says an engine may carry the first load message on the launch line (`Launch.SkillLoadOnLaunch`), whose turn shuttle waits for in the events file instead of typing, and its segment-color paragraph says the color follows that turn and precedes the retry; the run-flow comments of `start` and `loadSkillsThenPrompt` match.

**ImpactSummary:** Shuttle's start sequence for a skills launch reorders around an optional new `Launch` field; `loadSkillsThenPrompt`, `settleLoadTurn` and `loadSkillTurn` are re-signed with `start`, `loadSkillsThenPrompt` and `settleLoadTurn` their only callers, and an engine that leaves the field false keeps today's behavior.

**Commit:** 6: Wait for a launch-carried skill-load turn instead of typing it
