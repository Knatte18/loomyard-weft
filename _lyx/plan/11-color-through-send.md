# Card 11 — The segment color goes through the verified send after readiness

**Create:**
- `internal/shuttleengine/claudeengine#Claude.ColorCommand`
- `internal/testkit/shuttlefake#Engine.ColorCommand`

**Edit:**
- `internal/shuttleengine#Engine`
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Runner.TypeColor`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/testkit/shuttlefake#Engine`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/claudeengine/color_test.go`
- `internal/shuttleengine/claudeengine/prepare_test.go`
- `internal/testkit/shuttlefake/shuttlefake_test.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#Run.newSendContext`
- `internal/shuttleengine#Runner.newSendContext`
- `internal/shuttleengine#ErrSessionBusy`
- `internal/shuttleengine#ErrSubmissionNotLanded`

**Intent:**
Implement the two decisions added after Discussion on the segment color: every path that types the color command into a session plays it only after the provider is ready and through the verified send of cards 3 and 4 (idle wait, settled box, confirmed landing), so a color command never sits unsent in the input box.
Start's raw `playInputs(…, ColorSequence(…))` and `Runner.TypeColor`'s raw play are the two such paths.

`Engine` gains `ColorCommand(color segmentcolor.Color) string`: the text that sets a live session's display color as one submitted turn, or "" for a provider without one or a color it has no name for.
It is an interface method, so it carries no re-sign arrow; it lives on the seam because the command string is provider grammar (`PATTERN-shuttle-provider-seam`).
`Claude.ColorCommand` returns `/color <name>` from `colorNames`, "" outside the palette.
`shuttlefake.Engine` gains `ColorCommandFn` and `ColorCommand`, answering "" by default like its sibling sequences; `fakes_test.go`'s `fakeEngine` answers `COLOR:<color>`, or "" under `NoColorInputs`.

`Runner.start`, after `awaitStartup` and before `loadSkillsThenPrompt`, sends a non-empty `ColorCommand(strand.Color)` with `sendVerified(run.newSendContext(), command)` in place of the raw play, under the unchanged `strand.Color != "" && !spec.ColorByCaller` condition.
A failed color send, `ErrSessionBusy`, `ErrSubmissionNotLanded` or any other, is logged at Warn with the strand and color and never fails the launch, since the color is display only.

The color send needs an idle session, which a launch whose prompt rides its command line never offers before Start returns: the agent's first turn is already running, so the send would hold Start for the whole `send_ready_timeout_s` window and then drop the color as busy.
`Claude.Prepare` therefore defers the prompt pointer for a spec that shuttle colors, not only for one that names skills: when `len(spec.Skills) > 0` or `spec.Segment != "" && !spec.ColorByCaller`, the launch line carries no pointer and the pointer comes back as `Launch.PromptLine`.
Start then runs color, skill load (none for a skill-less spec) and prompt delivery in that order on a session that took no turn yet, and `loadSkillsThenPrompt` delivers the prompt through the verified send as for a spec with skills.
A segment that reed resolves to no color only defers the prompt, which `loadSkillsThenPrompt` still delivers at once.
`Launch.PromptLine`'s field comment says the engine defers the pointer for the color as well as the skills.
A spec with no segment, or with `ColorByCaller`, keeps the pointer on its line as today.
`Runner.TypeColor` resolves the run with `FindRun` as today and returns `sendVerified(r.newSendContext(state), command)` for a non-empty command, typing nothing for "".
Its doc comment says it waits for an idle session and confirms the landing like every verified send; its one production caller, `orchcli`'s `runnerSession.TypeColor` for the orch watcher, already logs a failure and goes on.
`ColorSequence` is left without callers here and deleted by card 12.

Bound (this card routes a display-only command through a send that can wait and clear): the color send can only delay the launch or fail, and a failure is logged and never ends a start or an orch tick.
The delay is capped by `send_ready_timeout_s` plus `submit_confirm_timeout_s` and the send's tails, as for any verified send; Enters and the clear go only into a box holding the color command, so no foreign draft or running turn is touched.
An unlanded color command is cleared from the box by card 4's clear, so the skill-load and prompt sends that follow find the box empty instead of failing busy against it.
Deferring the prompt of a colored skill-less spec moves its delivery into start's verified send, so a delivery that does not land now tears the run down as not started (card 5) instead of being left to the command line; every production spec that names a segment also names skills, so today only a hand-built spec takes that path.

`doc.go`'s segment-color paragraph says start sends `Engine.ColorCommand` through the verified send after readiness and that `Runner.TypeColor` does the same, and its skill-loading paragraph says an engine defers the prompt pointer for a colored spec too; `claudeengine/doc.go`'s palette paragraph names `ColorCommand` and drops the paced two-step sequence, and its "A spec that names skills starts on an empty input box" sentence widens to a spec shuttle colors; `Claude.Prepare`'s doc comment widens the same way.

Tests extend the existing color and deferral tests; no test function is added.
`run_test.go`'s `TestRunner_Start_TypesTheSegmentColorAfterStartup` keeps its rows, its "failing color play" row becoming a failing color send that still starts the run, and gains three rows: a skill-less colored spec whose engine defers its prompt, typing the color and then the prompt with no load turn; and, where the engine has the idle reading, a color command typed only once the pane reads idle, and a color command whose box never empties (`ErrSubmissionNotLanded`, cleared) still followed by the skill load into an empty box.
`session_test.go`'s `TestRunner_TypeColor_PlaysTheEngineSequenceOnALiveShuttleStrand` is a sequence of assertions, not a table; it is restructured into a table and renamed `TestRunner_TypeColor_SendsTheColorCommandOnALiveShuttleStrand`, its two existing cases (the command sent through the verified send on a live strand, an unknown guid refused with reed untouched) becoming rows beside one new row where a busy pane fails `ErrSessionBusy` with no key typed.
`claudeengine/color_test.go`'s `TestColorSequence_TypesTheColorCommandForEveryPaletteColor` is renamed `TestColorCommand_NamesEveryPaletteColor` and pins `ColorCommand` over the palette and an off-palette color instead, and the file's header comment names `ColorCommand` in place of `ColorSequence`'s choreography.
`claudeengine/prepare_test.go`'s `TestPrepare_SkillsDeferPromptPointer` is renamed `TestPrepare_DeferredPromptPointer`, and its loop over specs gains two specs beside the skill-less and skill-naming ones: a skill-less spec with a segment, which defers the pointer, and the same spec with `ColorByCaller`, which keeps it on the line; its doc comment says so.
`shuttlefake_test.go`'s default and scripted assertions name `ColorCommand` in place of `ColorSequence`.

**ImpactSummary:** `Engine` gains one method, implemented by `Claude` and both test fakes; start's color step and `Runner.TypeColor`, whose one production caller is the orch watcher's session, now wait for idle and confirm the landing; `Claude.Prepare` defers the prompt of a colored spec, which changes no production launch since every production spec with a segment already names skills.

**Verify:** go test -tags tmux ./internal/loomcli/... ./internal/shuttleengine/...

**Commit:** 11: shuttle: send the segment color through the verified send after readiness
