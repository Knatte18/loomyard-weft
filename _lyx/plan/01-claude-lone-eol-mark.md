# Card 1 — Claude reads a lone `%` in its input box as an empty box

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#isShellEOLMark` -> `func isShellEOLMark(interior []string) bool`
- `internal/shuttleengine/claudeengine/eolmark_test.go`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#boxInteriorChars`
- `internal/shuttleengine/claudeengine#IdleInputBoxFixture`
- `internal/testkit/shuttlefake#Reed`

**Intent:** Decisions D1 and D2: a box whose only content is zsh's end-of-line mark reads as empty, so a ready pane showing it takes the startup sends.

`shellEOLMark` is the one character `%`, with a doc comment naming it zsh's `PROMPT_EOL_MARK`, which the pane's login shell can leave on the row Claude later draws its box on.
`isShellEOLMark` reports whether the box interior's only non-blank content is exactly `shellEOLMark`: it strips the runes in `boxInteriorChars` and whitespace from every interior line, as `isBlankBoxInterior` does, and the concatenated remainder must equal `shellEOLMark`.
`%` beside any other character, on the same line or another, or any other lone character, is not the mark.

`Claude.IdleSession` reports idle when every interior line is blank, as today, or when `isShellEOLMark(interior)` holds.
`Claude.InputBoxText` returns `("", true)` when `isShellEOLMark(interior)` holds, and otherwise behaves as today.
Both doc comments say a lone `%` reads as empty and why.
The change lives in `claudeengine` only (`PATTERN-shuttle-provider-seam`), and every caller of the reading gets it: shuttle's pre-write guard and send confirmation, the startup sends and the orch watcher.

What it lets through, and the bound: the guard now admits a box holding the one character `%` alone and nothing else.
An operator draft of exactly `%`, typed outside `operator_quiet_s`, reads as empty and a paste merges with it; the operator-activity hold still runs before every guarded write.
The session idle probe's fallback for a strand with no run reads the box with no activity hold, so there a lone `%` reads idle with no operator check.

`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty; an operator draft of exactly a lone `%` typed outside `operator_quiet_s` then reads as empty and a write can merge with it.

Tests:
- `TestIdleSession` gains rows built from a rule, caret line and rule like its existing inline rows: a box holding `❯ %` reads idle; a box holding `❯ % done` and one holding `❯ #` read not idle; a boxed-side-bar box holding `│ ❯ %   │` reads idle.
- `TestInputBoxText` gains rows: `❯ %` answers `("", true)`; `❯ % done` answers `("% done", true)`; `❯ #` answers `("#", true)`.
- `eolmark_test.go`, package `claudeengine_test`, holds `TestStart_LoneEOLMarkBoxTakesTheStartupSends`.
  It drives `shuttleengine.NewRunner(...).StartGated` with the real classifier: an engine type embedding `*claudeengine.Claude` that overrides only `Prepare`, returning a `Launch` with a non-empty `PromptLine`, so every capability the guard reads (`Startup`, `IdleSession`, `InputBoxText`, `PasteSubmits`, `ColorSequence`, `SkillLoadMessage`, `ClassifySkillLoad`) is Claude's own.
  A `shuttlefake.Reed` answers `AddStrandFn` with a strand carrying a segment color Claude names, `StatusFn` with that strand live on a pane, and `CapturePaneFn` with `claudeengine.IdleInputBoxFixture` whose caret line carries `%` after the caret, on every capture, so the box shows the lone mark before and after every paste.
  The runner's `Config` is the corpus replay's (`PollIntervalMS 1000`, `LivenessEveryNPolls 1`, `RunTimeoutMin 60`, `StartupTimeoutS 30`, `BackgroundShellWaitMin 10`) plus `SendReadyTimeoutS 60` and `SubmitConfirmTimeoutS 30`, so no window or deadline closes before the sends.
  The runner gets a fake clock through `SetClock` whose `Sleep` advances `Now`, and the spec names one skill with a short `SkillLoadTimeout`, so the skill-load turn, which never ends, is skipped at its timeout without real waiting.
  The test asserts `StartGated` returns a non-nil run and a nil error, and that the reed's paste calls carry the `/color` command, the skill-load message for the skill and the prompt line, in that order.
  Before this card, the guard reads the `%` as a draft, the color send fails with a warning and the skill-load send fails busy, so `StartGated` returns an error wrapping `shuttleengine.ErrNotStarted`.
- `cmd/lyx/testdata/test-budget.yaml`: the `internal/shuttleengine/claudeengine` row rises by one for the new top-level test.

**ImpactSummary:** `Claude.IdleSession` and `Claude.InputBoxText` change their answer only for a box holding a lone `%`; every caller reads them through shuttle's `SessionCycler` and `InputBoxReader` capabilities, so no signature or caller changes.

**Commit:** 1: Claude reads a lone % in its input box as an empty box
