# Card 1 — Claude reads a lone `%` in the input box as an empty box

**Create:**
- `plan:internal/shuttleengine/claudeengine#isLoneEOLMark` -> `func isLoneEOLMark(interior []string) bool`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`

**Uses:**
- `internal/shuttleengine/claudeengine#boxInteriorChars`

**Intent:**
Decisions D1 and D2: a ready Claude pane whose input box shows only zsh's end-of-line mark `%` no longer fails every guarded send as a draft (issue #549).

`isLoneEOLMark`, beside `isBlankBoxInterior` in `session.go`, reports whether the box interior's only non-blank content, after every line is stripped of `boxInteriorChars` and whitespace, is the single character `%`, counted across all interior lines together.
`IdleSession` answers true for such a box, after its running-turn check and before its per-line blank check, so a running turn over a lone `%` still reads not idle.
`InputBoxText` answers `("", true)` for it.
Any other content stays a draft: `%` beside other text, two `%` characters, or any other single character.
The knowledge lives only in `claudeengine` (`PATTERN-shuttle-provider-seam`), so shuttle's pre-write guard, the startup color and skill-load sends and the orch watcher all read the box through these two methods and need no change.
`IdleSession`'s doc comment names the mark and why it is read as empty: the pane's login shell prints it before reed's launch line starts Claude, and a resize can leave it on the row Claude draws its box on, so Claude never received it as input.

Bound: the one character `%` alone in the box, nothing else; the operator-activity hold still runs before every write behind the pre-write guard.
The session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.
What this lets through: an operator draft of exactly `%`, typed outside `operator_quiet_s`, reads as empty, and a paste merges with it.

`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet, in the same commit: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty; an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, then reads as empty and a write can merge with it.

Tests, as rows in the existing tables:
- `TestIdleSession`: a ready box holding only `%` after the caret reads idle; a box holding `% done` reads not idle; a box holding a single `x` reads not idle; a running turn over a box holding only `%` reads not idle.
- `TestInputBoxText`: a box holding only `%` answers `("", true)`; a box holding `50%` answers `("50%", true)`; a box holding a single `x` answers `("x", true)`.

The start-level criterion, a start over a ready box with a lone `%` passing the color and skill-load sends, rests on these rows: Claude's `Startup` classifies on the caret alone, so the lone `%` never affected readiness, and shuttle's guarded sends judge the box only through `IdleSession` and `InputBoxText`, whose send-path behaviour existing `shuttleengine` start tests already cover over fake engines.

**ImpactSummary:** Two exported reading methods on `Claude` change for one input shape, a box holding only `%`; every caller (shuttle's pre-write guard, startup sends, `SessionIdle`, the orch watcher) inherits it through the `SessionCycler` and `InputBoxReader` seams with no edit, and no other provider changes.
