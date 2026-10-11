# Card 1 — Claude reads a lone `%` in the input box as an empty box

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#loneEOLMark` -> `func loneEOLMark(interior []string) bool`
- `internal/shuttleengine/claudeengine/eolmark_start_test.go`
- `plan:internal/shuttleengine/claudeengine_test#TestStart_LoneEOLMarkBoxTakesTheStartupSends` -> `func TestStart_LoneEOLMarkBoxTakesTheStartupSends(t *testing.T)`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine` -> `type eolMarkEngine struct`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.ContextTokens` -> `func (e *eolMarkEngine) ContextTokens(turnEnd shuttleengine.Event) shuttleengine.ContextReading`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.CompactedSince` -> `func (e *eolMarkEngine) CompactedSince(turnEnd shuttleengine.Event, since time.Time) (shuttleengine.CompactionBoundary, bool)`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.IdleSession` -> `func (e *eolMarkEngine) IdleSession(capture string) bool`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.PaneTooShort` -> `func (e *eolMarkEngine) PaneTooShort(capture string) bool`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.ClearSessionSequence` -> `func (e *eolMarkEngine) ClearSessionSequence() []shuttleengine.PaneInput`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.CompactSessionSequence` -> `func (e *eolMarkEngine) CompactSessionSequence(focus string) []shuttleengine.PaneInput`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.ReloadPluginsSequence` -> `func (e *eolMarkEngine) ReloadPluginsSequence() []shuttleengine.PaneInput`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.InputBoxText` -> `func (e *eolMarkEngine) InputBoxText(capture string) (string, bool)`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.SubmitSettle` -> `func (e *eolMarkEngine) SubmitSettle() time.Duration`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.TypeSequence` -> `func (e *eolMarkEngine) TypeSequence(text string) []shuttleengine.PaneInput`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.PasteSubmits` -> `func (e *eolMarkEngine) PasteSubmits() bool`
- `plan:internal/shuttleengine/claudeengine_test#eolMarkEngine.OwnPastePlaceholder` -> `func (e *eolMarkEngine) OwnPastePlaceholder(boxText, text string) bool`
- `plan:internal/shuttleengine/claudeengine_test#stepClock` -> `type stepClock struct`
- `plan:internal/shuttleengine/claudeengine_test#stepClock.Now` -> `func (c *stepClock) Now() time.Time`
- `plan:internal/shuttleengine/claudeengine_test#stepClock.Sleep` -> `func (c *stepClock) Sleep(d time.Duration)`

**Uses:**
- `internal/shuttleengine/claudeengine#isBlankBoxInterior`
- `internal/shuttleengine/claudeengine#boxInteriorChars`

**Intent:** Decision D1 and D2: Claude's input-box reading counts a box whose only non-blank content is a lone `%` as empty.

In `internal/shuttleengine/claudeengine/session.go`, add the constant `shellEOLMark` (`"%"`, documented as zsh's `PROMPT_EOL_MARK`, which the pane's login shell prints before reed's launch line starts Claude and which a resize can leave on the row Claude draws its box on).
Add `loneEOLMark(interior []string) bool`: it strips, from every interior line, the characters `boxInteriorChars` names and whitespace, joins what is left, and reports whether the result is exactly `shellEOLMark`.
`Claude.IdleSession` returns true for an interior `loneEOLMark` accepts, after its running-turn check and its box location, before the per-line `isBlankBoxInterior` loop.
`Claude.InputBoxText` returns `("", true)` for an interior `loneEOLMark` accepts.
Every other content stays a draft: `%` beside other text, `%%`, and any other single character.
Update both methods' doc comments to state the lone-`%` reading.

Bound: the one character `%` alone in the box is all this lets through.
The operator-activity hold still runs before every shuttle write behind the pre-write guard.
The session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.
A real operator draft of exactly `%`, typed outside `operator_quiet_s`, reads as empty and a paste merges with it.
The knowledge stays in `claudeengine` (`PATTERN-shuttle-provider-seam`); `shuttleengine` changes nothing.

`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty, so an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, reads as empty and a write can merge with it.
One sentence per line.

Tests.
`TestIdleSession` gains rows built from its `rule` string: a ready box holding `❯ %` reads idle (true); a box holding `❯ 50% done` reads not idle; a box holding `❯ %%` reads not idle; a box holding `❯ x` reads not idle; a boxed side-bar box `│ ❯ %   │` reads idle.
`TestInputBoxText` gains rows: `❯ %` reads `("", true)`; `❯ 50% done` reads `("50% done", true)`; `❯ %%` reads `("%%", true)`; `❯ x` reads `("x", true)`.

`eolmark_start_test.go` (package `claudeengine_test`) pins the start-level criterion.
`eolMarkEngine` embeds `*shuttlefake.Engine` and holds a `*claudeengine.Claude` from `claudeengine.New()`, and its methods above delegate one to one to that Claude, so the engine carries the real `SessionCycler`, `InputBoxReader` and `TextPaster` readings and no `SessionSignalParser`.
The embedded fake's `StartupFn` is `claude.Startup` and `ColorSequenceFn` is `claude.ColorSequence`.
Its `PrepareFn` records the run directory and returns a `Launch` with a `PromptLine`.
Its `SkillLoadMessageFn` returns `LOAD:` followed by the comma-joined skills, its `ParseEventsFn` reads a `STOP` line as one `shuttleengine.EventStop` carrying the line as `Raw`, and its `ClassifySkillLoadFn` reports the requested skills verified and loaded.
`stepClock` advances its `Now` by every `Sleep`.
The reed is a `shuttlefake.Reed` whose `AddStrandFn` returns strand `strand-1` with `Color` `cyan`, whose `Strands` lists that strand live, whose `CapturePaneFn` always returns a ready Claude pane whose box holds `❯ %` above the `? for shortcuts` footer, and whose `PasteSubmitFn` appends a `STOP` line to the run's `events.jsonl` when the pasted text starts with `LOAD:`.
`TestStart_LoneEOLMarkBoxTakesTheStartupSends` builds a runner over these with `shuttleengine.NewRunner` under the config `PollIntervalMS: 1000, LivenessEveryNPolls: 1, RunTimeoutMin: 60, StartupTimeoutS: 30`, `SetClock` and a spec naming one skill, calls `StartGated`, and asserts a non-nil run, a nil error, and the reed's `PasteSubmitCalls` texts equal, in order, the color sequence's text, the skill-load message and the prompt line.
Without D1 the pre-write guard reads the `%` box as a draft: the color paste never goes out and the skill-load send fails busy.

`cmd/lyx/testdata/test-budget.yaml` raises the `internal/shuttleengine/claudeengine` row by one for the new top-level test.

**ImpactSummary:** Two exported Claude readings change for one input shape; every caller (shuttle's pre-write guard and startup sends, `Runner.SessionIdle`, the orch watcher) inherits it through the engine seam, and no signature changes.

**Commit:** 1: Claude reads a lone % in the input box as an empty box
