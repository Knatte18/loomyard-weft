# Card 3 — planparser pattern entry lines

**Create:**
- `plan:internal/planparser#PatternEntryLine` -> `type PatternEntryLine struct`
- `plan:internal/planparser#CardPatternEntries` -> `func CardPatternEntries(card Card) []PatternEntryLine`
- `plan:internal/planparser#TestCardPatternEntries` -> `func TestCardPatternEntries(t *testing.T)`

**Edit:**
- `internal/planparser#checkPatternEntryLineCap`

**Uses:**
- `internal/planparser#codeSpans`

**Intent:**
D1 and D4 read a card's fenced verbatim `PATTERN.md` entry lines; `internal/planparser` stays their sole reader (PATTERN-sole-parsers), so the scan `pattern-entry-line-cap` already does becomes one exported function.

`PatternEntryLine` holds `ID`, the entry ID without backticks such as `PATTERN-cwd-resolution`, and `Line`, the whole line as written with any trailing `\r` trimmed.
`CardPatternEntries(card)` returns, in card text order, one entry per line inside a fenced code block of `card.Text` (the `spelling` spans `codeSpans` returns) that opens with ``- `PATTERN-``, the line shape `patternEntryLinePrefix` names; inline spans and prose are never read.
It keeps every line, a repeated ID included: last-wins is the projection's rule, not this reader's.

`checkPatternEntryLineCap` is rewritten over `CardPatternEntries`, with the same findings, order and detail text as today.

`TestCardPatternEntries`, table-driven in package `planparser`, builds `Card` values with `Text` set: a fenced entry line read with its ID; two fenced lines for one ID both returned in order; an inline-span entry line and a prose line ignored; a line under a `~~~` fence and one in an unclosed fence read; a CRLF line returned without its `\r`; a card with no fence returning nil.
The existing `pattern-entry-line-cap` rows keep passing unchanged and pin the rewrite.

**ImpactSummary:** one unexported check rewritten over the new exported reader; its finding text is unchanged, so no test or caller changes.
