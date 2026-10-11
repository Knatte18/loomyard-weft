# Card 1 — claude-lone-percent-box

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#holdsOnlyEOLMark` -> `func holdsOnlyEOLMark(interior []string) bool`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `internal/shuttleengine/claudeengine/doc.go`
- `pattern/PATTERN-pane-text-injection.md`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#isBlankBoxInterior`
- `internal/shuttleengine/claudeengine#boxInteriorChars`

**Intent:** Claude's input-box reading counts a box whose only non-blank content is a lone `%` as empty (D1, issue #549), and the pane-text-injection background states it (D2).

In `session.go`, `shellEOLMark` names zsh's `PROMPT_EOL_MARK`, the `%` the pane's login shell prints before Claude starts and a resize can leave on the row Claude draws its box on.
`holdsOnlyEOLMark` takes the interior `inputBoxInterior` returns and reports true when exactly one interior line, with the `boxInteriorChars` runes and whitespace stripped, equals `shellEOLMark`, and every other interior line is blank by `isBlankBoxInterior`.
`IdleSession` keeps its running-turn check and its per-line blank loop, and also reports idle when `holdsOnlyEOLMark` holds for the interior.
`InputBoxText` returns `("", true)` when `holdsOnlyEOLMark` holds, and its text otherwise as before.
Any other content stays a draft: `%` beside other text on the line or on another line, `%%`, and any other lone character such as `#`.
The comment on `holdsOnlyEOLMark` states the bound below in one sentence per line.

What the change lets through, and its bound: a box holding exactly one `%` and nothing else reads empty, so the pre-write guard, the startup sends and the orch watcher all treat it as free.
An operator draft of exactly `%` typed outside `operator_quiet_s` reads empty and a paste merges with it; the operator-activity hold still runs before every guarded write.
The session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.
Nothing else is let through.
The knowledge stays in `claudeengine` per `PATTERN-shuttle-provider-seam`; `shuttleengine` gains no `%`.

Tests, as rows in the existing tables:
`TestIdleSession` gains the rows "a lone percent after the caret reads idle" (`rule + "\n❯ %\n" + rule + "\n  ? for shortcuts\n"`, want true), "a lone percent inside boxed side bars reads idle" (want true), "a lone percent on the line under an empty caret line reads idle" (`rule + "\n❯ \n  %\n" + rule + …`, want true, since the record bounds the box's only non-blank content, not its line), "a percent beside other text is a draft" (`❯ % ls`, want false), "a percent on a second line under a draft is a draft" (want false) and "another lone character is a draft" (`❯ #`, want false).
`TestInputBoxText` gains the rows "a lone percent reads as empty text" (want `("", true)`), "a percent beside other text reads that text" (`❯ % ls`, want `("% ls", true)`) and "another lone character reads that character" (`❯ #`, want `("#", true)`).
The start-level acceptance criterion, a start whose ready box shows a lone `%` passing the color and skill-load sends, follows from these rows: shuttle's guarded sends read Claude's box only through the `SessionCycler` and `InputBoxReader` seams these two methods implement, and no shuttle-package test runs the real Claude engine, so a start-level test would need a new real-engine harness for no behavior these rows leave uncovered.

`doc.go` in `claudeengine` gains one sentence beside the `ClearInputSequence`/`OwnPastePlaceholder` paragraph: the box reading counts a lone `%`, the shell's end-of-line mark left on screen, as an empty box.
`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty; so an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, reads as empty and a write can merge with it.
The `PATTERN.md` entry line is unchanged.

**ImpactSummary:** Two exported methods of `claudeengine`'s `Claude` change their answer for one capture shape only; their callers (shuttle's pre-write guard, the startup sends, the orch watcher, the session idle probe) need no edit, and no signature changes.
