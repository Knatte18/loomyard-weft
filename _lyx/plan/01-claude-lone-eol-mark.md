# Card 1 — Claude reads a lone `%` in the input box as an empty box

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#holdsOnlyShellEOLMark` -> `func holdsOnlyShellEOLMark(interior []string) bool`
- `internal/shuttleengine/claudeengine/start_test.go`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#isBlankBoxInterior`
- `internal/shuttleengine/claudeengine#IdleInputBoxFixture`
- `internal/shuttleengine/claudeengine/testdata/corpus/skill-load-turn-end/events.jsonl`

**Intent:**
D1 and D2.
In `session.go`, `shellEOLMark` is the zsh end-of-line mark, `%`, that a pane's login shell can leave on the row Claude later draws its input box on.
`holdsOnlyShellEOLMark` reports whether the box interior's only non-blank content is that single character: every interior line has the caret marker and the `│` side bars stripped and is trimmed, and the non-blank lines joined with no separator equal `shellEOLMark` exactly.
`IdleSession` reads such an interior as idle: it returns true when every interior line is blank per `isBlankBoxInterior` or `holdsOnlyShellEOLMark(interior)` is true, and the running-turn check above it is unchanged.
`InputBoxText` returns `("", true)` for such an interior.
Any other content stays a draft: `%` beside other text on the same line or on another line, `%%`, and any other single character (`#`, `$`, `x`) read exactly as today.
The doc comments of both methods and of the new helper say why the mark is read as empty: Claude never received it as input, so the box is empty in fact and only its rendering carries the mark.

What this lets through, and the bound: only a box whose whole content is the one character `%`.
The pre-write guard's operator-activity hold still runs before every write, so an operator draft of exactly `%` reads as empty only when typed outside `operator_quiet_s`, and a paste then merges with it.
The session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.
Nothing else changes: every caller of the reading (shuttle's pre-write guard, the startup sends and the orch watcher) gets the same answer, and no Claude knowledge enters `shuttleengine` (`PATTERN-shuttle-provider-seam`).

Tests.
`TestIdleSession` gains rows built from the existing `rule` string with the box line `❯ %`: a box holding only `%` is idle (true); `❯ % foo` is not idle; `%` on its own line under a `❯ hello` caret line is not idle; `❯ %%` is not idle; `❯ #` is not idle.
`TestInputBoxText` gains rows: a box holding only `%` reads `("", true)`; `❯ % foo` reads `("% foo", true)`; `❯ #` reads `("#", true)`.
Each row fails against today's code or pins the bound the change must keep.

`start_test.go` is a new file in the external test package `claudeengine_test` holding one test, `TestStart_ReadyBoxHoldingShellEOLMark`: a start whose pane shows a ready box holding a lone `%` passes the color send and the skill-load send and returns a run.
It builds a `shuttleengine.Runner` through `shuttleengine.NewRunner` with a `shuttlefake.Reed` and an engine type declared in the test file that embeds `*claudeengine.Claude` (from `claudeengine.New()`) and overrides three methods, each to keep the test off the operator's home directory.
`Prepare` returns `shuttleengine.Launch{Cmd: "replay", SessionID: "replay", PromptLine: "Read the prompt."}` and records `filepath.Join(runDir, "events.jsonl")` as the events path, the way `replayCorpusCase` in `corpus_test.go` does.
`ProcessLiveness` returns `shuttleengine.LivenessAlive`, so the readiness reading never consults `~/.claude/sessions`.
`ClassifySkillLoad` returns the unverified `shuttleengine.SkillLoadReport{}`, so the load turn is confirmed without following the Stop line's `transcript_path`.
Every other method, `Startup`, `IdleSession`, `InputBoxText`, `ColorSequence`, `SkillLoadMessage`, `ParseEvents` and `ParseSessionSignals` included, is Claude's own.
Each guarded send's readiness reading is then ready, or unknown before the events file holds a signal, in which case `busyReading` falls back to the pane: `Startup` reads the capture ready and `IdleSession` decides.
Either way the box reading decides the send, and an operator-activity query on the reed fake answers not active.
The reed fake's `AddStrandFn` returns a strand with a segment color set; `StatusFn` reports the strand live with a pane id; `CapturePaneFn` always returns the ready capture: a rule line, `❯ %`, a rule line and the `? for shortcuts` footer, the shape `IdleInputBoxFixture` has with `%` after the caret.
`PasteSubmitFn` records every pasted text and, when the text is the skill-load message, appends the first line of `testdata/corpus/skill-load-turn-end/events.jsonl` (the skill-load Stop) to the events file, so the load turn ends.
The spec carries that case's skills (`scribe:prose`, `scribe:conversation`, `ly:board`), a temp output file and a timeout; the runner's clock is a small clock type in the test file whose `Sleep` advances `Now` without waiting (`PATTERN-test-speed`).
The test asserts `StartGated` returns a non-nil run and a nil error, and that the pasted texts are, in order, the text of Claude's color command for the strand's color, the skill-load message and `Read the prompt.`.
Against today's code the guard reads `%` as a draft: the color send fails busy (only a Warn) and the skill-load send fails busy, so `StartGated` returns an error wrapping `ErrNotStarted` and the test fails.

`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty; an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, then reads as empty and a write can merge with it.
The line is one sentence per line (`scribe:prose`).

The new top-level test raises `internal/shuttleengine/claudeengine` in `cmd/lyx/testdata/test-budget.yaml` by one.

**ImpactSummary:** Two exported reads in claudeengine widen by one exact input; every caller (the pre-write guard, startup sends, orch watcher, session idle probe) inherits it through the `InputBoxReader` and `SessionCycler` seams with no signature change.

**Commit:** 1: Claude reads a lone % in its input box as an empty box
