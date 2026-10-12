# Card 1 — claude-lone-eol-mark-reads-empty

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#boxText` -> `func boxText(interior []string) string`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`

**Delete:**
- `internal/shuttleengine/claudeengine#isBlankBoxInterior`
- `internal/shuttleengine/claudeengine#boxInteriorChars`

**Prosa:**
- `pattern/PATTERN-pane-text-injection.md`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#gateCaretMarker`

**Intent:** Decisions D1 and D2 of the decision record: Claude's input-box reading treats a box whose only content is a lone `%` as empty, and the pane-text-injection background records what that lets through.

`shellEOLMark` is the one character the reading forgives, with a doc comment saying it is zsh's `PROMPT_EOL_MARK`, printed by the pane's login shell before reed's launch line starts Claude, which a resize can leave on the row Claude later draws its box on.
Claude never received it as input, so the box is empty in fact and only its rendering carries the mark.

`boxText` is the one interior reading both callers share: for each interior line it removes `gateCaretMarker` and the side bar `│` (the two characters `InputBoxText` strips today), trims the line, and joins the non-blank lines with a single space, exactly as `InputBoxText` does today.
When the joined text equals `shellEOLMark`, it returns the empty string.
Anything else, `%` beside other text or any other single character included, is returned as text.

`InputBoxText` locates the box through `inputBoxInterior` as today and returns `boxText(interior)`.
`IdleSession` keeps its running-turn check and box location, then reports idle when `boxText(interior)` is empty, in place of the per-line `isBlankBoxInterior` loop.
`isBlankBoxInterior` and `boxInteriorChars` lose their only caller and are deleted.
The one reading `boxInteriorChars` stripped that `boxText` does not is a bare `>`: `inputBoxInterior` finds the box by `gateCaretMarker` alone, so a `>` between the rules is never the caret, and under the record's acceptance criterion any single character other than `%` reads as a draft.
The `IdleSession` doc comment states the lone-mark rule and that an operator draft of exactly `%` typed outside reed's quiet window reads as empty, so a write behind the pre-write guard can merge with it; nothing else is let through, and the operator-activity hold still runs before every guarded write.
The session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check; the comment says so.

Tests, as rows in the two existing tables, each built from the `rule` string the tables already use:
`TestIdleSession` gains a lone `%` after the caret (idle), a lone `%` inside the boxed side-bar shape (idle), `%` beside other text such as `% ls` (not idle), and a single other character such as `x` (not idle);
`TestInputBoxText` gains a lone `%` (empty text, ok true), `% ls` (`% ls`, ok true) and a single `x` (`x`, ok true).

`pattern/PATTERN-pane-text-injection.md` gains one sentence under the pre-write guard bullet, kept to one sentence per line: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty; an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, reads as empty and a write can merge with it.
The `%` knowledge stays inside `claudeengine`, per `PATTERN-shuttle-provider-seam`; `shuttleengine` is untouched.

**ImpactSummary:** Two methods of Claude's session reading change and two unexported helpers go, all inside claudeengine; every caller of IdleSession and InputBoxText (shuttle's pre-write guard, the startup sends, the orch watcher) sees a lone `%` as an empty box and nothing else changes.
