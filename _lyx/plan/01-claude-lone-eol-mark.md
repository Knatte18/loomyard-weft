# Card 1 — claude-lone-eol-mark

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/shuttleengine/claudeengine#loneEOLMark` -> `func loneEOLMark(interior []string) bool`
- `plan:internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxPassesStartupSends` -> `func TestStart_LoneEOLMarkBoxPassesStartupSends(t *testing.T)`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#boxInteriorChars`
- `internal/testkit/shuttlefake#Reed`

**Intent:**
Claude's input-box reading treats a box holding only a lone `%` as an empty box (decision D1, issue #549).
The `%` is zsh's `PROMPT_EOL_MARK`, printed by the pane's login shell before Claude starts and left on the row Claude later draws its box on, so Claude never received it as input.

`loneEOLMark` reports whether the interior lines of the box, with the characters of `boxInteriorChars` and all whitespace stripped from every line, hold exactly the one character `%` and nothing else.
`Claude.IdleSession` reads such a box as empty: where it now returns false on the first non-blank interior line, it returns false only when the interior is not blank and `loneEOLMark` is false.
`Claude.InputBoxText` returns `("", true)` when `loneEOLMark` holds, and its text unchanged otherwise.
Both doc comments state the lone-`%` reading and why.
Any other content stays a draft: `%` beside other text on the same or another interior line, and any other single character such as `#`.

Bound: the one character `%` alone in the box is all this lets through.
The operator-activity hold still runs before every write behind shuttle's pre-write guard.
The session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.
An operator draft of exactly `%`, typed outside `operator_quiet_s`, reads as empty and a paste can merge with it.
The knowledge stays in `claudeengine` (`PATTERN-shuttle-provider-seam`); `shuttleengine` gains nothing.

`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet (decision D2): the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty;
an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, then reads as empty and a write can merge with it.
One sentence per line.

Tests:
- `TestIdleSession` gains rows on the inline `rule` capture shape: a box holding `❯ %` reads idle; `❯ % ls`, `❯ #`, and `❯ %` with a second interior line holding text each read not idle.
- `TestInputBoxText` gains rows: `❯ %` reads `("", true)`; `❯ % ls` reads `("% ls", true)`; `❯ #` reads `("#", true)`.
- `TestStart_LoneEOLMarkBoxPassesStartupSends` drives `shuttleengine.NewRunner` with `claudeengine.New()` as the engine and a `shuttlefake.Reed` whose `CapturePaneFn` always answers a ready Claude screen whose box holds `❯ %` (the `pane-idle-empty.txt` shape with `%` after the caret), whose `StatusFn` answers the strand live, and whose `AddStrandFn` returns a strand carrying a segment color.
  The spec names two skills, so the launch defers its prompt to `Launch.PromptLine`.
  Set `Config.RunDir` to a test temp dir so the test can find the run's `events.jsonl`; each `PasteSubmitFn` call that carries the skill-load message appends one Claude `Stop` hook line to it, shaped as in `testdata/skillload-mixed.jsonl`, so each load turn ends.
  The runner's clock is a test clock that advances on `Sleep`, so nothing waits out real time.
  It asserts `StartGated` returns a non-nil run and a nil error, and that the pasted texts include the engine's color command and `SkillLoadMessage` for the two skills.
  Before the change the color send and the skill-load send each fail the guard as busy on the `%`.
- Raise the `internal/shuttleengine/claudeengine` row of `cmd/lyx/testdata/test-budget.yaml` by one for the new top-level test.

**ImpactSummary:** Narrows Claude's empty-box reading to admit a lone `%`, which every caller of `IdleSession` and `InputBoxText` (shuttle's pre-write guard, the startup sends, the orch watcher) inherits; no signature changes.

**Commit:** 1: Claude reads a lone % in the input box as an empty box
