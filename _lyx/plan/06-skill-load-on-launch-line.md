# Card 6 — skill-load-on-launch-line

**Create:**
- `plan:internal/shuttleengine#Run.typeSegmentColor` -> `func (run *Run) typeSegmentColor(color segmentcolor.Color)`
- `plan:internal/shuttleengine#skillReed.AddStrand` -> `func (r *skillReed) AddStrand(spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/shuttleengine#skillReed.playLoad` -> `func (r *skillReed) playLoad(list string) error`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#maxLaunchPromptBytes`
- `internal/shuttleengine/claudeengine#TestPrepare_SkillsDeferPromptPointer`
- `internal/shuttleengine/claudeengine#TestPrepare_RejectedBeforeArtifacts`
- `internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxTakesStartupSends`
- `internal/shuttleengine#hookStateSkillEngine`
- `internal/shuttleengine#hookStateSkillEngine.IdleSession`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.loadSkillsThenPrompt` -> `func (run *Run) loadSkillsThenPrompt(promptLine string, color segmentcolor.Color) (Result, error)`
- `internal/shuttleengine#Run.settleLoadTurn`
- `internal/shuttleengine#Run.loadSkillTurn` -> `func (run *Run) loadSkillTurn(loader SkillLoader, skills []string, timeout time.Duration, typed bool) (turnEnd Event, ended, died bool, err error)`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.SendText`
- `internal/shuttleengine#skillReed.Status`
- `internal/shuttleengine#skillStartFixtureWith`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine#TestRunner_Start_TypesTheSegmentColorAfterStartup`
- `internal/shuttleengine/doc.go`
- `PATTERN.md`
- `docs/overview.md`
- `contracts/stencils/roleskills_test.go`

**Intent:**
Decision D2: a spec that names skills launches with the engine's skill-load message as the launch argument, so the first load turn is never typed.

Engine (`claudeengine`, per `PATTERN-shuttle-provider-seam`): `Claude.Prepare`, for a spec with skills, sets the launch argument to `c.SkillLoadMessage(spec.Skills)` in place of the empty one and still returns the prompt pointer as `Launch.PromptLine`, for a fresh and a resumed launch alike.
It refuses, before writing any artifact, a load message longer than `maxLaunchPromptBytes`, naming the length, the limit and the skill count, as it refuses an over-long pointer.
`maxLaunchPromptBytes`'s comment says the bound now covers the pointer and the skill-load message.
The comment inside `Claude.Prepare` that says a spec naming skills "starts on an empty input box" is reworded to say it launches with the skill-load message and returns the pointer as `Launch.PromptLine`.
A spec without skills launches with the pointer, unchanged.

Shuttle: `Launch.PromptLine`'s doc says a spec with skills launched with the engine's skill-load message as its first turn and the pointer comes back here.
`Runner.start`, after `awaitStartup`, branches on `spec.Skills`:
- without skills it calls `typeSegmentColor(strand.Color)`, then `loadSkillsThenPrompt(launch.PromptLine, "")` only when the prompt line is set, as today;
- with skills it calls `loadSkillsThenPrompt(launch.PromptLine, strand.Color)`.
`typeSegmentColor` is the color step moved out of `start` unchanged: it plays the color through `typeColor` and the verified send when the color is set and `spec.ColorByCaller` is false, and a failed play only warns.
`loadSkillsThenPrompt` settles the first load turn untyped, then types the color, then retries the missing skills typed, then sends the prompt pointer, each typed send through the verified send.
A first load turn that times out skips its skills with a warning and the color and prompt still follow; a pane that dies during it ends the start died through `abandonStartup` before any color is typed.
`settleLoadTurn` passes `typed = final` to `loadSkillTurn`: the first turn is launch-carried, the retry is typed; its doc says so.
`loadSkillTurn` sends the load message through `sendVerified` only when `typed` is true; otherwise it only polls the events file from the run's offset for the turn end.
Bound: the launch-carried load message skips the verified send's landing check; its landing is proven by a turn end in the events file instead, and a turn that never starts or never ends is bounded by the skill-load timeout (`Spec.SkillLoadTimeout`, else the engine's `DefaultSkillLoadTimeout`), measured from the end of `awaitStartup`, after which its skills are skipped with a warning.
The first load turn is classified from its turn end through `ClassifySkillLoad` exactly as a typed one.

Docs in the same commit: `PATTERN.md`'s `PATTERN-role-skills-typed` entry reads "the spawning module names it on the launch spec and lyx delivers it, on the launch line or typed; no stencil asks an agent to load a skill." keeping the entry id and `(test)` mark; `docs/overview.md`'s skills-and-parent-directive line says the same; the header comment of `contracts/stencils/roleskills_test.go` follows; `internal/shuttleengine/doc.go`'s color and skill-loading paragraphs and `internal/shuttleengine/claudeengine/doc.go`'s skill paragraph state the launch-carried first turn and the new order.

Test fixture: `skillReed` gains `LaunchLoad string`, the list the launch carries, and `FirstLoadStatusDelay int`.
`skillReed.playLoad` is today's per-load reaction factored out of `SendText` (count the turn, die, hang, or append `STOP:loaded <list>`).
`skillReed.AddStrand` delegates to the embedded reed, then plays turn 0 for `LaunchLoad` when it is set: at once, or, with `FirstLoadStatusDelay` set, on that many `Status` calls later (the `Status` override plays the deferred turn 0, whichever of end, hang or die its script names).
A turn 0 scripted to die always plays deferred, `FirstLoadStatusDelay` defaulting to one more `Status` call than `awaitStartup`'s first ready tick takes, so the pane dies during the load turn and not during startup.
`skillReed.SendText` plays the typed loads after turn 0, and records whether the events file already held a turn end when a `COLOR:` text was typed.
`skillStartFixtureWith` sets `LaunchLoad` to the spec's skills joined with commas.
`hookStateSkillEngine`'s doc comment and `IdleSession` drop the clause about a typed first skill load: the first load is no longer typed or captured, so the pane the test leaves busy is one the pane fallback refuses until the hook state gets past it, and `IdleSession`'s `LOAD:` match stays only for a typed retry.

Rows:
- `TestPrepare_SkillsDeferPromptPointer`: with skills, `Launch.Cmd` carries the shell-quoted `SkillLoadMessage` of the skills and no pointer, and `PromptLine` is the pointer, fresh and resumed; without skills, unchanged.
- `TestPrepare_RejectedBeforeArtifacts`: a row whose skill list makes the load message exceed `maxLaunchPromptBytes` is refused naming the limit, with no artifacts written.
- `TestStartup_SkillLoadTurns`: every row's typed sequence loses its first `LOAD:`; "every skill loads" types only `do the task`; the retry rows type `LOAD:b|do the task`; the timeout row hangs turn 0, types `do the task`, skips both skills and takes at least the timeout; the died row dies on turn 0, deferred past startup as above, and types nothing, ending with an error wrapping `ErrNotStarted` and no pane tail; the hook-state row is renamed to say a session start alone delivers the prompt into a pane the pane fallback refuses.
- `TestRunner_Start_TypesTheSegmentColorAfterStartup` pins the order "color after the load turn ends": the colored rows type `COLOR:orange|do the task`; a new row with a missing skill types `COLOR:orange|LOAD:b|do the task`; a new row with `FirstLoadStatusDelay` set asserts the color is still typed, after the turn end reached the events file, and before the prompt; a new row whose first load turn hangs, with a short `SkillLoadTimeout`, types `COLOR:orange|do the task`, so the color and prompt follow a timed-out first turn; a new row with no skills, whose `configure` clears `PrepareLaunch.PromptLine` as a skill-less launch has none, types `COLOR:orange` and nothing else, which pins the no-skill branch's color through `typeSegmentColor`.
The table gains a `hangs []int` field passed to `skillStartFixtureWith`, and its `spec` builds `Skills` from a per-row field so the no-skill row can leave it empty.
- `TestStartup_SendFailuresTearDown`: the test builds its own `skillReed` rather than going through `skillStartFixtureWith`, so it sets `LaunchLoad` on it for a skill row, which plays turn 0 at launch.
  All three skill rows, "a skill-load send that stays busy", "a skill load held past the send window by an active operator" and "a prompt-offset persist that fails", name skills `a` and `b` with a first report missing `b`, so the typed retry `LOAD:b` is the send that stays busy, is held, or breaks the record through `brokenRunRecordReed`; the row table gains that `reports` field.
- `TestStart_LoneEOLMarkBoxTakesStartupSends` (`claudeengine/start_test.go`): the first load is launch-carried and never pasted, so its wanted pastes become `/color blue` and the prompt line, not `SkillLoadMessage`; nothing ends the launch-carried load turn, which times out on the test's virtual clock before the color and prompt go out, and the test's comment says so.
- `TestStartup_NoSkillsNoPromptLineSendsNothing` keeps pinning that a run without skills types nothing at start.

**ImpactSummary:** Changes the launch line of every skilled Claude spawn and the order of start's typed sends (color moves after the first load turn); the only callers of the re-signed `loadSkillsThenPrompt` and `loadSkillTurn` are `Runner.start` and `settleLoadTurn` in `run.go`; tests and four docs follow, no other package's code changes.
