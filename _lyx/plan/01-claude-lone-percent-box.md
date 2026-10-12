# Card 1 — Claude reads a lone % in the input box as empty

**Create:**
- `plan:internal/shuttleengine/claudeengine#isLoneEndOfLineMark` -> `func isLoneEndOfLineMark(interior []string) bool`
- `internal/shuttleengine/claudeengine/start_lone_mark_test.go`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D1 and D2.
`isLoneEndOfLineMark` takes the interior lines `inputBoxInterior` returns and reports whether exactly one line is non-blank under `isBlankBoxInterior`'s rule and, with the caret, side bars and surrounding whitespace removed, that line is the single character `%`.
`IdleSession` treats such an interior as blank, so a ready box holding only `%` reads idle; its running-turn check and its box location are unchanged.
`InputBoxText` returns empty text with `ok` true for it.
Any other content stays a draft: `%` beside other text on the same line or on another line, `%%`, and any other single character such as `#`.
Only `claudeengine` learns the `%` (`PATTERN-shuttle-provider-seam`); `shuttleengine` gains no Claude specifics.
What it lets through: one `%` alone in the box is no longer a draft, so the pre-write guard's third fact (box empty and readable) passes for it.
The operator-activity hold still runs before every write behind the guard, which bounds the change for a pane an operator is typing into.
The session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.
A real operator draft of exactly `%` typed outside `operator_quiet_s` reads as empty and a paste merges with it; nothing else is let through.
No refusal or guard branch is added: the change narrows what an existing reading admits.

Tests, all extending existing tables except the one new file:
- `TestIdleSession` gains rows: a box holding only `%` after the caret is idle; a box holding only `%` between side bars is idle; `% hello` is not idle; `%` on one line with text on another is not idle; a box holding only `#` is not idle.
- `TestInputBoxText` gains rows: a lone `%` box answers `("", true)`; `% hello` answers `("% hello", true)`; a lone `#` box answers `("#", true)`.
- `start_lone_mark_test.go` is an external-package (`claudeengine_test`) file with one new top-level test, `TestStart_LoneEndOfLineMarkBox`, covering the acceptance criterion no existing test reaches: a `Runner.Start` over `shuttlefake.Reed` and a thin wrapper embedding `claudeengine.New()` that overrides only `Prepare` (to return a fixed `Launch` with a `PromptLine` and skip settings writing), on a pane whose capture is a ready Claude screen whose input box holds only `%`.
  The strand returned by the fake reed carries a color, so the color send and the skill-load or prompt send both cross the pre-write guard.
  The runner gets an advancing fake clock through `Runner.SetClock` and a `shuttleengine.Config` with one-second `send_ready_timeout_s` and `submit_confirm_timeout_s`, as `corpus_test.go` does with `SetClock` and an explicit `Config`, so no send waits out a real interval.
  The fake reed's paste hook appends the turn-start line the Claude hook would write to the run's events file, so the prompt send lands.
  It asserts the fake reed recorded a paste-submit for the color command and for the prompt, that `Start` returns a run, and that no error wraps `ErrSessionBusy`.
  The skill-load send is not given its own row: `settleLoadTurn` and the prompt delivery both go through `sendVerified`, the one guarded path, so the prompt send exercises the same guard.
  A same-shape row with `% hello` in the box asserts the guard holds the prompt send and the error wraps `ErrSessionBusy`, which is what keeps the first row's pass from being vacuous.
  `Start` resolving through `ErrSubmissionNotLanded` after the paste (the static fake pane never shows the text land) is not what this test asserts.
- `cmd/lyx/testdata/test-budget.yaml` raises the `internal/shuttleengine/claudeengine` row by one for the new top-level test.

`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty; an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, reads as empty and a write can merge with it.
`PATTERN.md`'s own entry line for `PATTERN-pane-text-injection` is unchanged.

**ImpactSummary:** `IdleSession` and `InputBoxText` feed shuttle's pre-write guard, the startup sends and the orch watcher, so all three see a lone `%` as an empty box; every other box content classifies as before.
