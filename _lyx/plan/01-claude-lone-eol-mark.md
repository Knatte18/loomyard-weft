# Card 1 — Claude reads a lone `%` in the input box as an empty box

**Create:**
- `internal/shuttleengine/claudeengine#shellEOLMark`
- `internal/shuttleengine/claudeengine#holdsOnlyEOLMark`
- `internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxTakesStartupSends`
- `internal/shuttleengine/claudeengine/start_test.go`
- `internal/shuttleengine/claudeengine/testdata/pane-idle-eol-mark.txt`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine/session_test.go`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#isBlankBoxInterior`
- `internal/shuttleengine/claudeengine#gateCaretMarker`
- `internal/shuttleengine/claudeengine/corpus_test.go`
- `internal/shuttleengine/claudeengine/testdata/pane-idle-empty.txt`

**Intent:**
Decisions D1 and D2 (issue #549): a ready Claude pane whose input box shows only a stray `%` stops failing every shuttle start.

In `session.go`, add `shellEOLMark`, documented as zsh's `PROMPT_EOL_MARK`, which the pane's login shell prints before reed's launch line starts Claude and which a pane resize can leave on the row Claude later draws its box on, so Claude never received it as input.
Add `holdsOnlyEOLMark(interior)`: it strips `gateCaretMarker`, the `│` side bar and whitespace from every interior line and reports whether what remains, across all lines together, is exactly `shellEOLMark`.
It strips the same characters `InputBoxText` strips, not `boxInteriorChars`, so the two readings agree on a box holding `>%`.

`Claude.IdleSession`: after the running-turn check and after `inputBoxInterior` succeeds, a box for which `holdsOnlyEOLMark` is true reads idle.
`Claude.InputBoxText`: a box for which `holdsOnlyEOLMark` is true returns `("", true)`.
Every other content stays a draft: `%` beside other text, any other single character, and a lone `%` outside a located box (no caret line or no rule) are unchanged.
Each doc comment gains one sentence stating the lone-`%` reading and its bound.

Bound, stated in both doc comments: this lets through one thing, an operator draft of exactly `%` typed outside reed's quiet window, which reads as empty so a paste can merge with it.
The operator-activity hold still runs before every write behind the pre-write guard;
the session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.

Tests, in `session_test.go`:
- `TestIdleSession` gains rows: the new `pane-idle-eol-mark.txt` fixture reads idle; a boxed side-bar box holding only `%` (`"│ ❯     %   │"`) reads idle; a box holding `100% done` reads not idle; a box holding a lone `#` reads not idle.
- `TestInputBoxText` gains the same four captures, reading `""`, `""`, `"100% done"` and `"#"`, each with ok true.

`testdata/pane-idle-eol-mark.txt` is a copy of `pane-idle-empty.txt` whose caret line carries a lone `%` after the caret.

`start_test.go` holds `TestStart_LoneEOLMarkBoxTakesStartupSends`, the end-to-end check through shuttle's real start path.
Wire it as `corpus_test.go` does: a `shuttleengine.NewRunner` over a `shuttlefake.Reed`, with `SetClock` on an injected clock that never sleeps for real.
The engine is a test type embedding `*Claude` that overrides only `Prepare` to return a canned `Launch` with a `PromptLine`, so every box and session reading is Claude's own.
The reed's `AddStrandFn` returns a strand with a segment color, and its capture serves `pane-idle-eol-mark.txt` on every read.
So each send's pre-write guard, the skill-load send's included, reads the lone-`%` box as idle, and each paste's confirm read sees an empty box, which lands the send.
The spec names one skill.
The test asserts that `StartGated` returns a run and a nil error, that the color command and the skill-load message each reached the pane through the fake's paste or text hook, and that no send failed busy or in-draft.
It is the only `claudeengine` test that drives a start with Claude's own box reading, so it raises `internal/shuttleengine/claudeengine` in `cmd/lyx/testdata/test-budget.yaml` from 103 to 104.

`pattern/PATTERN-pane-text-injection.md`: under the "Shuttle's pre-write guard" bullet, add one indented line after "A write the guard holds fails wrapping `ErrSessionBusy` and writes nothing, and the caller retries later.":

```markdown
  The provider's box reading defines empty, and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty, so an operator draft of exactly `%` typed outside `operator_quiet_s` reads as empty and a write can merge with it.
```

**ImpactSummary:** Claude's two box readings, used by shuttle's pre-write guard, the startup sends and the orch watcher, read a lone `%` as an empty box; no other content changes reading.

**Commit:** 1: Claude reads a lone % in the input box as an empty box
