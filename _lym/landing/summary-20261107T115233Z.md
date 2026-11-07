# Comment lint: flag new or changed // comment lines at 120 characters

`lym loom lint-comments` and the gate's `comments` lint step checked only where a `//` comment line breaks.
They now also enforce the line-length rule `scribe:prose` sets under "Line breaks":
a comment line's text stays under 120 characters.

## What changes

- `internal/commentlint` gains `linelength.go` with `LineLengthLimit` (120) and the check.
  It counts characters, not bytes, of the text after `//`, without the line's indentation or a list item's marker.
  A line at the limit or over it is a finding; one character under passes.
- Only new or changed lines are checked: a line whose text stands unchanged in a comment block of the base is skipped,
  so legacy long lines are flagged only once an edit touches them.
- Exempt, as the skill words it: headings, blockquotes (`>`), table rows (`|`),
  and a line holding a URL, code span or other unbroken word that alone reaches the limit.
  Directive comments, end-of-line comments and generated files are not checked, as for the break check.
- `Finding` gains a `Rule` field (`wrapped break` or `line length`) and a `String` method rendering `file:line: rule: text`.
  The `lint-comments` envelope, the gate's failure report and the verify gate's findings all print that form,
  and their way-forward text now covers splitting an over-long line.
- A line-length finding fails the gate exactly as a break finding does: the same callers, the same exit paths.

## Design decisions

- The skill also says where to split an over-long line: at a semicolon, a colon or a comma before a clause-opening conjunction, or by rewriting.
  That choice is not mechanical; `doc.go` states it as a bound, and the existing break check judges the break a split leaves.
- "Changed" is decided per line by exact text, not by the break check's word-pair carry-over:
  any edit to a long line makes it a finding, which is the rule the board entry asks for.
- Rendering the rule into each finding line changes the printed form of break findings too (`wrapped.go:3: wrapped break: // ...`);
  the two integration tests that pin that text are updated.

## Tests

`TestFindNewLongLines` (tier 0) covers the limit boundary, what is counted, multi-byte text, carry-over versus edit, each exemption,
and the lines never checked. `TestFindingString` pins the rendered form.
