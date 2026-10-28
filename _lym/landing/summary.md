# Comment lint carries over a legacy break whose lines only rename words

The comment lint in `internal/commentlint` called a line break new when the word pair across it was absent from the base text.
A word rename inside a legacy fixed-column-wrapped paragraph changes the word pair at every break of that paragraph, so each break became a finding although the wrapping was never touched.
A mechanical binary rename run halted at its producer row on several hundred such findings.

A break is now also carried over when the base holds a consecutive line pair that matches the break's two lines apart from words substituted in place.
The base pair must have the same word count on each line as the break's lines, and the unchanged words must outnumber the substituted ones across the two lines.
A rename inside a legacy wrapped paragraph therefore flags none of its breaks.

Closes #881

## Design decisions

- The substitution match runs only after the semantic-break check and the word-pair lookup have both failed, so most breaks are settled by the cheap checks.
  Base line pairs are indexed by their two word counts, which limits the match to pairs that can match at all.
- The threshold is a strict majority of unchanged words.
  A line pair with exactly half its words unchanged is still flagged, and a test row pins that boundary.
- A reflow that moves a word across a line end changes both lines' word counts, so it does not match the pair it came from and is still flagged.
- A rename that changes a line's word count, such as one word replaced by two, is not covered and still flags the breaks around that line.
- A short line pair with a rename on each line has no unchanged majority and is still flagged.
- `baseLineEndPairs` becomes `newBaseBreaks`, which returns the word pairs and the line-pair index together; `carriesOver` is the one place that decides whether a break was already there.
- `doc.go` describes the new carry-over rule.

## Tests

The new cases are rows in the tier-0 `TestFindNewWrappedBreaks` table:
a rename of a line's last word, a rename of a line's first word, a new break beside a rename, a line pair rewritten in most of its words, and the exactly-half boundary.
Every existing row keeps its result.
No test run is recorded here; run `lym gate test ./internal/commentlint` before landing.
