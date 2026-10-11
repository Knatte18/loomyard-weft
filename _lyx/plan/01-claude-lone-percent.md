# Card 1 — Claude reads a lone `%` in the input box as empty

**Create:**
- `plan:internal/shuttleengine/claudeengine#loneEndOfLineMark` -> `func loneEndOfLineMark(interior []string) bool`
- `plan:internal/shuttleengine/claudeengine#TestStart_LoneEndOfLineMarkBox` -> `func TestStart_LoneEndOfLineMarkBox(t *testing.T)`
- `plan:internal/shuttleengine/claudeengine#preparedClaude` -> `type preparedClaude struct`
- `plan:internal/shuttleengine/claudeengine#preparedClaude.Prepare` -> `func (c preparedClaude) Prepare(runDir string, spec shuttleengine.Spec, cfg shuttleengine.Config) (shuttleengine.Launch, error)`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `internal/shuttleengine/claudeengine/doc.go`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#boxInteriorChars`
- `internal/shuttleengine/claudeengine#IdleInputBoxFixture`

**Intent:**
D1 and D2 of the decision record.
`loneEndOfLineMark` takes the interior lines `inputBoxInterior` returns and reports true when, with every rune of `boxInteriorChars` and all whitespace removed from every line, exactly the single character `%` remains.
`IdleSession` returns true for such a box once the running-turn check has passed, and `InputBoxText` returns `("", true)` for it; every other box reads as before.
A `%` beside other text, `%%`, and any other single character stay a draft, and a capture with no box stays not idle and `ok` false.
Only claudeengine learns the `%`, so `PATTERN-shuttle-provider-seam` holds and `shuttleengine` gains no Claude specifics.

What this lets through: a box whose only content is `%` reads empty, so a guarded write can paste behind it, and an operator draft of exactly `%` typed outside `operator_quiet_s` merges with the paste.
The operator-activity hold still runs before every write behind the pre-write guard; the session idle probe's fallback for a strand with no run reads the box with no activity hold, so a lone `%` reads idle there with no operator check.
The bound is the single character `%` alone in the box; nothing else is let through, with the caret and side-bar runes the existing `boxInteriorChars` set strips (`>` included, as `isBlankBoxInterior` already treats it) counting as no content.

`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty, and an operator draft of exactly a lone `%` typed outside `operator_quiet_s` reads as empty and a write can merge with it.
The same commit adds one sentence to `claudeengine/doc.go` next to the input-box reading, and updates the `IdleSession` and `InputBoxText` doc comments.

Tests, all red before the code:
- `TestIdleSession` gains rows for `❯ %` between rules (idle), the same box in boxed side-bar form (idle), `❯ % more text` (not idle) and `❯ x` (not idle).
- `TestInputBoxText` gains the same shapes: a lone `%` reads `("", true)`, `% more text` reads that text, `x` reads `x`.
- `TestStart_LoneEndOfLineMarkBox` is the one new top-level test, because no existing claudeengine test drives a start.
  It lives in `session_test.go` (package `claudeengine`, so the handle's package matches) and drives `shuttleengine.NewRunner(...).StartGated` with `preparedClaude`, which embeds a real `*Claude` and overrides `Prepare` alone to return a canned `Launch` with a `PromptLine` and to record the run directory.
  Reed is `testkit/shuttlefake`'s `Reed`, whose `AddStrandFn` returns a strand with a color set, with `CapturePaneFn` answering `IdleInputBoxFixture` with a `%` after the caret, a `PasteSubmitFn` that appends a `Stop` hook line to the run's events file when the pasted text is the skill-load message, and a fake clock.
  The spec sets `Skills`, and the strand carries a color, so the color send, the skill-load send and the prompt send all pass the guard.
  It asserts `StartGated` returns a run and nil error, and that the color text, the skill-load message and the prompt line were each pasted once; before the change the guard holds the first send busy and none is pasted.
- `cmd/lyx/testdata/test-budget.yaml` raises the `internal/shuttleengine/claudeengine` row by one for that test.

**ImpactSummary:** `IdleSession` and `InputBoxText` are reached only through the `SessionCycler` and `InputBoxReader` seams by shuttle's pre-write guard, the start sends and the orch watcher; only a box holding exactly `%` changes reading, and no caller outside claudeengine names either method.

**Commit:** 1: claudeengine: read a lone % in the input box as empty
