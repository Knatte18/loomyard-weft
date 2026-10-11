# Card 1 — Claude reads a lone `%` in its input box as an empty box

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#eolMarkOnly` -> `func eolMarkOnly(interior []string) bool`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#isBlankBoxInterior`

**Intent:**
Decision D1 and D2 (issue #549).
A ready Claude pane whose input box shows only zsh's end-of-line mark `%` after the caret is an empty box in fact, and today every startup send into it fails as a draft.

`shellEOLMark` names the mark, with a comment saying it is zsh's `PROMPT_EOL_MARK`, printed by the pane's login shell before Claude starts and left on the row Claude later draws its box on, so Claude never received it as input.
`eolMarkOnly` reports whether the box interior's only non-blank content, after the caret, side bars and whitespace are stripped exactly as `isBlankBoxInterior` strips them, is the single string `shellEOLMark`: one interior line reads exactly `%` once stripped and every other line is blank.
`Claude.IdleSession` treats an interior for which `eolMarkOnly` holds as blank, so it reports idle when no turn runs.
`Claude.InputBoxText` returns `("", true)` for such an interior.
Any other content stays a draft: `%` beside other text on the line or on another line, `%%`, and any other single character, such as `#`.
Update both methods' doc comments to state the lone-`%` reading.

Bound: the one string `%` alone in the box, nothing else.
The operator-activity hold still runs before every write behind shuttle's pre-write guard.
The session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.
An operator draft of exactly `%` typed outside `operator_quiet_s` reads as empty and a paste can merge with it; that is the whole of what the change lets through.

Tests, as rows in the existing tables:
- `TestIdleSession`: a ready box holding only `%` after the caret classifies idle (`true`); the same box with `%` inside the boxed side bars classifies idle; a box holding `% ls` classifies not idle; a box holding `#` classifies not idle; a box holding `%` with a running-turn hint above classifies not idle.
- `TestInputBoxText`: a box holding only `%` reads `("", true)`; a box holding `% ls` reads `("% ls", true)`; a box holding `#` reads `("#", true)`.

`pattern/PATTERN-pane-text-injection.md`: add one line under the pre-write guard bullet stating that the provider's box reading defines "empty", that Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty, and that an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, therefore reads as empty and a write can merge with it.
The fix lives in `claudeengine` alone per `PATTERN-shuttle-provider-seam`; `shuttleengine` gains no `%` knowledge.

**ImpactSummary:** Claude's idle and box-text readings, read by shuttle's pre-write guard, the startup sends, the send verifier and the orch watcher, now read a lone `%` box as empty; every other reading is unchanged.

**Commit:** 1: Claude reads a lone % in its input box as an empty box
