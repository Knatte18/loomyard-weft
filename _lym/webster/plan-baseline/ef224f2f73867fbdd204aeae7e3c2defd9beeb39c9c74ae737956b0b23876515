# Card 2 — citation-extract

**Create:**
- `internal/discussionparser#Citation`
- `internal/discussionparser#Citations`
- `internal/discussionparser#TestCitations`

**Extends:**
- `internal/discussionparser/doc.go`

**Uses:**
- `internal/discussionparser#fencedLines`
- `internal/discussionparser#addedDecisionSpans`
- `internal/discussionparser#carryOverMarker`
- `internal/discussionparser#patternBulletSuffix`

**Intent:**
The parser half of D3 and D4: the record's sole parser extracts every citation the citation check tests, with its line and mark, and judges none of them, so `internal/discussionparser` stays stdlib-only.

`Citation` holds `Token string` (the text between one pair of single backticks, verbatim), `Line int` (1-based line of the record) and `Marked bool`.
`Citations` returns, in record order, one `Citation` per inline code span of a line: the text between an opening backtick and the next backtick on that line, a span with empty text skipped.
`Marked` is true when the closing backtick is followed immediately by ` (new)` or ` (removed)`.
It skips:
the lines inside fenced code blocks, as `fencedLines` marks them;
the lines of each `### Added after Discussion` entry, the spans `addedDecisionSpans` returns;
a `## PATTERN changes` bullet line ending with the suffix `patternBulletSuffix` matches;
and every line from a `<!-- lym:carry-over … -->` open marker to its `<!-- /lym:carry-over … -->` close marker, both read through `carryOverMarker`, an unclosed block running to the end of the record.
A heading line is read like any other line, so a backticked heading such as `## Decisions` comes back as a citation and the check in internal/loomshed classifies it.

`TestCitations`, a table in the existing tier0 test file of the package: a span's token and line come back; a span followed by ` (new)` and one by ` (removed)` are marked and one followed by ` (old)` is not; two spans on one line come back in order; a span inside a fenced block, inside an `### Added after Discussion` entry, on a suffixed PATTERN changes bullet and inside a carry-over block are absent, while the same span on an unsuffixed PATTERN changes bullet is present; an empty span is absent.

Docs: `internal/discussionparser/doc.go` names `Citations`, what it skips and that judging a citation is internal/loomshed's.

**ImpactSummary:** new exported parser surface with no caller until card 4, plus one doc paragraph; nothing existing changes.
