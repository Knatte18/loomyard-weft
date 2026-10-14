# Card 2 — The launch line carries the skill-load message

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#maxLaunchPromptBytes`
- `internal/shuttleengine/claudeengine#launchPointer`
- `internal/shuttleengine/claudeengine#buildLaunchCmd`
- `internal/shuttleengine/claudeengine/skillload.go`
- `internal/shuttleengine/claudeengine#TestPrepare_PromptLaunchLimit`
- `internal/shuttleengine/claudeengine#TestPrepare_SkillsDeferPromptPointer`
- `internal/shuttleengine/claudeengine#TestPrepare_RejectedBeforeArtifacts`
- `internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxTakesStartupSends`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#SkillLoader`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.loadSkillsThenPrompt` -> `func (run *Run) loadSkillsThenPrompt(promptLine string, color segmentcolor.Color) (Result, error)`
- `internal/shuttleengine#Run.settleLoadTurn`
- `internal/shuttleengine#Run.loadSkillTurn` -> `func (run *Run) loadSkillTurn(loader SkillLoader, skills []string, timeout time.Duration, typed bool) (turnEnd Event, ended, died bool, err error)`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.SendText`
- `internal/shuttleengine#skillReed.Status`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine#skillStartFixtureWith`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestRunner_Start_TypesTheSegmentColorAfterStartup`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine/doc.go`
- `PATTERN.md`
- `docs/overview.md`
- `contracts/stencils/roleskills_test.go`

**Create:**
- `plan:internal/shuttleengine#Run.typeSegmentColor` -> `func (run *Run) typeSegmentColor(color segmentcolor.Color)`

**Uses:**
- `internal/shuttleengine/claudeengine_test#TestCorpus_ReplaysEachTurnEnd`
- `internal/shuttleengine#TestStartup_NoSkillsNoPromptLineSendsNothing`

**Intent:**
Decision D2: a spec that names skills launches with the engine's skill-load message as the launch argument, in place of today's empty input box.
The first load turn is then never typed.

Engine side (`claudeengine`, per `PATTERN-shuttle-provider-seam`):
for a spec with skills, `Claude.Prepare` sets `launchArg` to `c.SkillLoadMessage(spec.Skills)` and keeps returning the prompt pointer as `Launch.PromptLine`.
A spec without skills launches with the pointer and an empty `PromptLine`, unchanged.
Before any artifact is written, beside the pointer's own limit check, `Prepare` refuses a skill-load message that is not one line or exceeds `maxLaunchPromptBytes`.
"Not one line" means a skill name holding `\n` or `\r`; each error names the skill or the message's length and the limit.
`Prepare`'s doc comment and `claudeengine/doc.go`'s skill-loading paragraph say the launch line carries the load message and the pointer comes back as `Launch.PromptLine`.
The paragraph's "one typed message" becomes "one message, carried on the launch line for the first turn and typed for the retry".
Comments the change makes false are corrected in the same commit:
- `maxLaunchPromptBytes`: the skill-load message can reach the bound too.
- `launchPointer`: the pointer is the argument a session without skills starts with.
- `buildLaunchCmd`: its `pointer` parameter carries the launch argument, the pointer or the load message.
- `skillload.go`'s file header: the first load message rides the launch line and the retry is typed.
- `TestPrepare_PromptLaunchLimit`'s doc: the skill-load limit is pinned in `TestPrepare_RejectedBeforeArtifacts`.

Shuttle side:
`Launch.PromptLine`'s field comment and `SkillLoader`'s doc comment state the contract.
An engine that realizes skills carries `SkillLoadMessage(spec.Skills)` as its launch argument and returns the prompt as `PromptLine`, so shuttle types no first load turn.
`Runner.start` resolves the segment color once, empty for no color or `spec.ColorByCaller`.
With a `PromptLine` it calls `loadSkillsThenPrompt(launch.PromptLine, color)`.
Without one it calls `typeSegmentColor(color)` after `awaitStartup`, as today.
`typeSegmentColor` is the moved color step: it types nothing for an empty color, and it logs a failed play at Warn without failing the launch.

`loadSkillsThenPrompt`, for a spec with skills, runs in this order:
1. Settle the first load turn without typing it: `loadSkillTurn(..., typed=false)` only polls the events file for its turn end.
2. Type the color.
3. Type the retry turn for missing skills, if any.
4. Deliver the prompt pointer, each typed step through the verified send.

Without skills it types the color, then delivers the prompt.
`settleLoadTurn` passes `typed = final`, since the first turn rides the launch line and only the retry is typed.
The first turn's timeout (`Spec.SkillLoadTimeout`, else `DefaultSkillLoadTimeout`) is measured from `loadSkillTurn`'s start, which is the end of `awaitStartup`.
A timeout skips its skills with a warning and the color and the prompt follow.
A pane death during it ends the startup died, through `abandonStartup` as today.
Bound: the launch-carried message skips the verified send's landing check, and its landing is proven instead by a turn end in the events file.
Every send that stays typed keeps using the verified send (`PATTERN-pane-text-injection`).

Docs in the same commit:
- `shuttleengine/doc.go`'s "Segment color" paragraph: the color goes out after the launch-carried load turn ends, before any retry.
- `shuttleengine/doc.go`'s "Skill loading" paragraph: the first load turn is carried on the launch line and awaited, and the retry is typed.
- `docs/overview.md`'s "skills and the parent directive" bullet: a spawn's skills are delivered by the provider engine before the prompt, the first load on the launch line.
- `contracts/stencils/roleskills_test.go`'s header comment: "lyx types it" becomes "lyx delivers it, on the launch line or typed".
- `PATTERN.md`: the entry is reworded in place.
  It keeps its byte length, because `PATTERN.md` sits within a few bytes of `lyx loom lint-pattern`'s headroom line:

```
- `PATTERN-role-skills-typed` — Loading a skill into a spawn: its module names it on the launch spec, and lyx delivers it on the launch line or typed; no stencil asks to load one. (test)
```

Test doubles:
- `skillStartFixtureWith` sets `PrepareLaunch.Cmd` to `"LOAD:" + strings.Join(spec.Skills, ",")` when the spec names skills, mirroring the real engine's launch line.
- `skillReed` plays the launch-carried turn as load turn 0 when the added strand's `Cmd` (last of `fakeReed.AddStrandCalls`) starts with `"LOAD:"`.
  It plays it at its first `CapturePane`, which is the startup probe's capture, so a settled turn still costs no virtual time.
  `Hangs[0]` and `Dies[0]` apply to it, and typed loads count on from turn 1.
- A new field `EndAfterPolls int`, when positive, appends turn 0's end at that many `Status` calls after the first capture instead of at it.
- A new field records, for each `"COLOR:"` typed, whether turn 0's end had already been appended.

Test rows:
- `TestStartup_SkillLoadTurns`: every row's `wantTyped` loses its leading first `LOAD:` entry.
  For example, "every skill loads in one turn before the prompt" becomes `"do the task"`, the retry rows `"LOAD:b|do the task"`, the never-ending first turn `"do the task"` at `atLeastTimeout`, and the dying first turn `""` with `wantDied`.
  The `hookState` row's name says the prompt, not the load, goes into a pane the pane fallback refuses once the launch-carried turn's end is recorded.
- `TestRunner_Start_TypesTheSegmentColorAfterStartup`: the doc comment says the color follows the launch-carried load turn.
  The rows expect `"COLOR:orange|do the task"` or `"do the task"`, and "ColoredStrandIsColoredBeforeTheSkillLoad" is renamed "ColoredStrandIsColoredAfterTheLaunchCarriedLoadTurn".
  It gains `skills` and `reports` fields and two rows:
  - "AMissingSkillsRetryFollowsTheColor": skills `a,b`, first report missing `b`; typed `"COLOR:orange|LOAD:b|do the task"`.
  - "ALongLaunchCarriedLoadTurnDelaysTheColorWithoutLosingIt": `EndAfterPolls: 3`; typed `"COLOR:orange|do the task"`, virtual elapsed above zero, and the color typed only after turn 0's end.
  - "ALaunchWithoutSkillsOrPromptLineIsColoredAfterStartup": no skills, and the `configure` callback clears `PrepareLaunch.PromptLine`, so `start` takes the no-`PromptLine` branch; typed `"COLOR:orange"`.
- `TestStartup_SendFailuresTearDown`: the skill rows set `PrepareLaunch.Cmd` to `"LOAD:a"` and gain a `missing` field.
  `missing` makes the first report miss `a`, so the typed retry is the send that stays busy, is held by an active operator, or (with `breakRecord`) breaks `run.json` through `brokenRunRecordReed`.
  The busy row's name says the retry load.
- `claudeengine`:
  - `TestPrepare_SkillsDeferPromptPointer`: with skills, `Launch.Cmd` contains `shell.ForGOOS().Quote(New().SkillLoadMessage(skills))` and not the pointer, and `PromptLine` is the pointer, for a fresh and a resumed launch.
    Without skills it is unchanged.
  - `TestPrepare_RejectedBeforeArtifacts` gains two rows: `skill_load_message_over_launch_limit` (one skill name of `maxLaunchPromptBytes` bytes, `wantErrContains: "launch limit"`) and `skill_name_with_line_break` (`"a\nb"`, `wantErrContains: "line break"`); the errors carry those phrases.
  - `TestStart_LoneEOLMarkBoxTakesStartupSends`: its canned launch carries no load turn, so the load turn times out on the virtual clock.
    It asserts the pane received `/color blue` and `Read the prompt.`, and never the load message.
- `TestCorpus_ReplaysEachTurnEnd`'s `skill-load-turn-end` case runs unchanged: its first line, the load turn end, is now read by the untyped wait instead of after a typed send.
- `TestStartup_NoSkillsNoPromptLineSendsNothing` keeps pinning that a run without skills types nothing at start.

**ImpactSummary:** Claude's launch line for a skilled spec now carries the load message; shuttle's start no longer types the first load turn and types the color after it; every in-package skill and color test, two claudeengine Prepare tests, the start test and the role-skills docs and PATTERN entry change with it.

**Commit:** 2: shuttle: carry the skill-load message on the launch line
