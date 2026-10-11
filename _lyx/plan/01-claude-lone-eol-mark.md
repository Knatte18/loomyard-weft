# Card 1 — claude-lone-eol-mark

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#isLoneEOLMark` -> `func isLoneEOLMark(interior []string) bool`
- `internal/shuttleengine/claudeengine/testdata/pane-idle-eol-mark.txt`
- `internal/shuttleengine/claudeengine/eolmark_start_test.go`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `internal/shuttleengine/claudeengine/doc.go`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Claude's input-box reading treats a box whose only non-blank content is a lone `%` as an empty box (decision D1, issue #549), and the pane-text-injection background records that narrowing in the same commit (D2).

`shellEOLMark` is zsh's `PROMPT_EOL_MARK`, which the pane's login shell prints before reed's launch line starts Claude and which a pane resize can leave on the row Claude later draws its box on.
`isLoneEOLMark` takes the interior lines `inputBoxInterior` returns, strips `boxInteriorChars` and whitespace from each, and reports whether the concatenation of what remains across all lines is exactly `shellEOLMark`.
`%` beside any other text, on the same line or another interior line, and any other single character, read false.
`Claude.IdleSession` reports idle when the running-turn hint is absent, the box is located, and either every interior line is blank under `isBlankBoxInterior` or `isLoneEOLMark` holds.
`Claude.InputBoxText` returns `("", true)` when `isLoneEOLMark` holds on the located interior, and otherwise reads the box as today.
The knowledge stays in `claudeengine` per `PATTERN-shuttle-provider-seam`; `shuttleengine` gains nothing.

Bound: the one character `%` alone in the box reads empty, and nothing else is let through.
The pre-write guard still runs its operator-activity hold before every write, so a lone `%` typed inside `operator_quiet_s` still holds the write.
What the change lets through is an operator draft of exactly `%` typed outside the quiet window, which reads empty so a paste merges with it, and the session idle probe's fallback for a strand with no run, which reads the box with no activity hold, so a lone `%` there reads idle with no operator check.

`pane-idle-eol-mark.txt` is `pane-idle-empty.txt` with the caret line reading `❯ %`.
`TestIdleSession` and `TestInputBoxText` gain rows: the fixture reads idle and its box text reads `("", true)`; a box holding `% foo`, a box holding `%` on the caret line and other text on a second interior line, and a box holding a lone `#` each read not idle and return their text.

`eolmark_start_test.go` (package `claudeengine_test`) adds one top-level test that starts a run through `shuttleengine.NewRunner` with the real `claudeengine.New()` engine over a `shuttlefake.Reed` whose `CapturePaneFn` always answers the `pane-idle-eol-mark.txt` capture, an injected clock, and a spec naming a segment color and one skill.
Its `PasteSubmitFn` records each paste and, for the skill-load message, appends a Claude Stop hook line to the run's events file so the load turn ends; a skill-load report that cannot read a transcript is confirmed unverified and never fails the start.
It asserts `StartGated` returns a run and nil error, and that the color command, the skill-load message and the prompt line were each pasted once.
Without the classifier change the color send's pre-write guard holds on the `%` box and the start fails `ErrNotStarted` wrapping `ErrSessionBusy`, so the test is seen failing before the classifier change lands.
The new top-level test raises the `internal/shuttleengine/claudeengine` row in `cmd/lyx/testdata/test-budget.yaml` by one.

`internal/shuttleengine/claudeengine/doc.go` gains one sentence beside the paste paragraph: a box holding only a lone `%`, the shell's end-of-line mark left on screen, reads empty to both `IdleSession` and `InputBoxText`.

`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty;
an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, then reads empty and a write can merge with it.

**ImpactSummary:** every caller of Claude's box reading (shuttle's pre-write guard, the startup sends, the session idle probe and the orch watcher) now reads a lone-`%` box as empty; no signature changes.

**Commit:** `1: Claude reads a lone % in the input box as an empty box`
