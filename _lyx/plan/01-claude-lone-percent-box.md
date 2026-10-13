# Card 1 — Claude reads a lone `%` in the input box as an empty box

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#isEOLMarkOnly` -> `func isEOLMarkOnly(interior []string) bool`
- `plan:internal/shuttleengine/claudeengine#TestStart_LonePercentBoxTakesTheStartupSends` -> `func TestStart_LonePercentBoxTakesTheStartupSends(t *testing.T)`
- `plan:internal/shuttleengine/claudeengine#lonePercentEngine` -> `type lonePercentEngine struct`
- `plan:internal/shuttleengine/claudeengine#lonePercentEngine.Prepare` -> `func (e *lonePercentEngine) Prepare(runDir string, spec shuttleengine.Spec, cfg shuttleengine.Config) (shuttleengine.Launch, error)`
- `plan:internal/shuttleengine/claudeengine#steppingClock` -> `type steppingClock struct`
- `plan:internal/shuttleengine/claudeengine#steppingClock.Now` -> `func (c *steppingClock) Now() time.Time`
- `plan:internal/shuttleengine/claudeengine#steppingClock.Sleep` -> `func (c *steppingClock) Sleep(d time.Duration)`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#isBlankBoxInterior`
- `internal/shuttleengine/claudeengine#boxInteriorChars`

**Intent:** Decisions D1 and D2: Claude's input-box reading treats a box whose only non-blank content is a lone `%` as empty, and the pane-text-injection background says so.

In `session.go`, `isEOLMarkOnly` reports whether the box interior's only non-blank content, after every rune of `boxInteriorChars` (the caret, side bars and `>`) and all whitespace are stripped from every interior line and the lines are joined, is exactly `shellEOLMark`.
Its doc comment names the mark as zsh's `PROMPT_EOL_MARK`, printed by the pane's login shell before reed's launch line starts Claude and left on the row Claude later draws its box on, so Claude never received it as input.
`Claude.IdleSession` keeps its running-turn and box-location checks; where the interior holds a non-blank line it now returns `isEOLMarkOnly(interior)` instead of false.
`Claude.InputBoxText` returns `("", true)` when `isEOLMarkOnly(interior)` holds, and otherwise builds its text exactly as today.
`%` beside any other text, on the same line or another interior line, and any other single character stay a draft.
The doc comments of both methods state the lone-`%` reading.

Bound (the guard this narrows): only the one character `%` alone in the box is let through.
Shuttle's pre-write guard still asks the operator-activity hold before every write, so the one thing the change admits is an operator draft of exactly `%` typed outside reed's quiet window, which then reads empty and a paste can merge with it.
The session idle probe's fallback for a strand with no run (`Runner.SessionIdle`) reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.
The `%` knowledge stays in `claudeengine` (`PATTERN-shuttle-provider-seam`); `shuttleengine` gains nothing.

`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty; an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, then reads as empty and a write can merge with it.

Tests:
- `TestIdleSession` gains rows over the inline `rule` captures: a box holding only `%` after the caret is idle (true); `%` alone on an interior line below an empty caret line is idle (true); the boxed side-bar box `│ ❯ %   │` is idle (true); `% done` after the caret is not idle (false); a lone `x` after the caret is not idle (false).
- `TestInputBoxText` gains rows: a lone `%` after the caret reads `("", true)`; `% done` reads `("% done", true)`; a lone `x` reads `("x", true)`.
- `TestStart_LonePercentBoxTakesTheStartupSends` (in `session_test.go`, package `claudeengine`) drives `shuttleengine.NewRunner` over a `shuttlefake.Reed` and a `lonePercentEngine`, which embeds `*Claude` from `New()` so every real capability (startup classification, box reading, idle reading, paste, color and skill-load sequences, event and signal parsing) is Claude's, and overrides only `Prepare` to return `shuttleengine.Launch{Cmd: "claude", SessionID: "s", PromptLine: "Read the prompt."}` without writing provider files.
  The reed's `AddStrandFn` returns a strand `{GUID: "g", PaneID: "%1", Color: segmentcolor.Orange}` and the test seeds `Reed.Strands` with that strand live; `CapturePaneFn` always returns a ready box whose caret line holds only `%`: a rule, `❯ %`, a rule and the `ReadyFooterFixture` footer.
  `runner.SetClock` installs a `steppingClock`, whose `Sleep` advances `Now` by the slept duration, so the send-ready, submit and skill-load windows run on virtual time; the config is `PollIntervalMS: 1000, LivenessEveryNPolls: 1, StartupTimeoutS: 30, RunTimeoutMin: 60` with the other send windows at their template values, and the spec carries `Skills: []string{"a"}` and `SkillLoadTimeout: time.Minute`, so the load turn, which no events line answers, is skipped at its timeout without failing the start.
  The test asserts `StartGated` returns a non-nil run and a nil error, and that `Reed.PasteSubmitCalls` texts are, in order, `/color orange`, `Claude.SkillLoadMessage([]string{"a"})` and `Read the prompt.`.
  Before D1 the color send and the skill-load send are held busy, since the pane fallback reads the `%` box as not idle, and the skill-load send's failure tears the start down with `ErrNotStarted`, so the test fails then.
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/shuttleengine/claudeengine` from 103 to 104 for the new top-level test.

**ImpactSummary:** Two `Claude` box readers change for one input shape; every caller of the reading (shuttle's pre-write guard, startup sends, the orch watcher's idle probe) inherits it with no signature change and no change in `shuttleengine`.

**Commit:** 1: Claude reads a lone % in the input box as an empty box
