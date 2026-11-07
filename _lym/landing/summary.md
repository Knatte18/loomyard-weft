# Comment lint: flag `//` comment lines of 120 characters or more

`lym loom lint-comments` and the gate's `comments` lint step checked only where a `//` comment line breaks.
Nothing bounded a line's length.
`internal/commentlint` now also enforces the line-length rule that `scribe:prose` sets under "Line breaks".
A finding fails the gate as the wrapped-break findings do.

## What changes

- A new or changed `//` comment line whose counted text is `LineLengthLimit` (120) characters or more is a finding.
  A line one character shorter passes.
- The count is in characters, not bytes, of the text after `//`, without indentation or a list item's marker.
- Exempt: a heading of any level, a blockquote (`>`), a table row (`|`), and a line holding a URL, code span or other unbroken word that alone reaches the limit.
- Not checked: directive comments, end-of-line comments and generated files.
- A line that stands unchanged in a comment block of the base text is skipped.
  Each occurrence in the base excuses one occurrence in the new text, so a second copy of a long line is flagged.
- `Finding` gains a `Rule` field (`wrapped break` or `line length`) and a `String()` method rendering `file:line: rule: text`.
  Findings of both rules are ordered by line within a file.
- The rendering moves to `Finding.String()` in `lym loom lint-comments`, the `lym gate test` failure report and the `loomshed` verify gate.
  Their finding lines now carry the rule name, and their way-forward text names both fixes.
- `docs/overview.md`, the gate template comment, and the `lint-comments` help text name both rules.

## Design decisions

- The rule's split advice (break at a semicolon, a colon or a comma before a clause-opening conjunction, never mid-phrase) is not checked.
  The wrapped-break check only judges the break a split leaves.
  The package's `doc.go` states this bound.
- The base comparison is on the line's trimmed literal, so reindenting a long line does not make it new.

## To check by hand

- Run `lym loom lint-comments` over a branch with an existing long comment line outside the diff, and confirm it is not reported.
