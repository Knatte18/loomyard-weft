# Card 1 — claude-lone-percent-reads-empty

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#boxInteriorText` -> `func boxInteriorText(interior []string) string`

**Prosa:**
- `pattern/PATTERN-pane-text-injection.md`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#isBlankBoxInterior`

**Intent:** D1 and D2 of the decision record: Claude's input-box reading treats a box whose only content is a lone `%` as empty, and the pane-text-injection background records what that lets through.

`shellEOLMark` is the one character, `%`, which zsh's `PROMPT_EOL_MARK` prints before reed's launch line starts Claude and which a resize can leave on the row Claude later draws its box on.
Its doc comment says so, and says that nothing in a shuttle pane types a lone `%` into the box.

`boxInteriorText` holds the strip-and-join `InputBoxText` does today over the interior lines (caret marker and side bars removed, each line trimmed, non-blank lines joined by one space), and returns the empty string when that joined text is exactly `shellEOLMark`.
`InputBoxText` returns `boxInteriorText(interior)` in place of its inline loop.
`IdleSession` keeps its per-line `isBlankBoxInterior` loop and, on the first non-blank interior line, returns `boxInteriorText(interior) == ""` instead of false, so a box whose only content is the lone mark classifies idle and any other content, `%` beside other text included, still classifies not idle.
Both doc comments gain one sentence naming the lone-mark reading and why it is empty in fact.

Bound: the one character `%` alone in the box, after the caret and side bars are stripped.
Nothing else is let through: the running-turn needle, the no-box case, the too-short case and the operator-activity hold in front of every guarded write are unchanged.
The input that reaches the new branch after every upstream check passes is a capture with an input box on screen, no running-turn hint, and interior text that strips to exactly `%`; `TestIdleSession`'s new row reaches it.
What the change lets through is an operator draft of exactly `%` typed outside `operator_quiet_s`: it now reads as empty and a paste can merge with it.

Tests, as rows of the existing tables, with the rule string the tables already build the box from:
`TestIdleSession` gains "a lone end-of-line mark after the caret" (true), "the mark beside a draft" (false) and "another lone character" (such as `#`, false);
`TestInputBoxText` gains "a lone end-of-line mark reads empty" (`""`, ok true) and "the mark beside a draft reads the draft" (the mark and the text, ok true).

`pattern/PATTERN-pane-text-injection.md` gains, under the pre-write guard bullet (the one beginning "Shuttle's pre-write guard stands in front of every such write"), two indented lines, one sentence each: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty;
an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, therefore reads as empty and a write can merge with it.
The `PATTERN.md` entry line is unchanged.

**ImpactSummary:** Two Claude classifier methods and one background file; every reader of the box (shuttle's pre-write guard, the startup sends, the orch watcher's idle probe) now reads a lone `%` as empty, with no change outside `claudeengine` and the pattern file.
