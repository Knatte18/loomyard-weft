# Comment lint ignores a legacy wrapped paragraph changed by a rename

The comment lint judged a line break new when the word pair across it was absent from the base text.
A word rename inside a legacy fixed-column-wrapped comment changes that pair wherever the renamed word sits at a line end or line start, so a mechanical rename across the repository produced hundreds of findings for wrapping it never touched, and exhausted a verify budget.

## What changes

`internal/commentlint` now collects the base text's breaks as `baseBreaks`: the word pairs across each line end, as before, plus each base line pair's words, indexed by the two lines' word counts.
A break that is no semantic break is still not new when `baseBreaks.carriesOver` finds a base line pair with the same word counts whose words equal the break's two lines position by position except for in-place substitutions, and whose unchanged words outnumber the substituted ones across the two lines.

A rename keeps every line end and word count, so its breaks carry over.
A reflow that moves a word across a line end changes both lines' word counts, so the break it creates is still flagged, as is a line pair rewritten in most of its words.

The package doc in `internal/commentlint/doc.go` states the new rule and its bound.

## Design decisions

- The match needs equal word counts per line, so a rename that replaces one word by several still flags the breaks around the changed line; the reported case renamed one token to one token.
- The threshold is a strict majority of unchanged words across the two lines; a short line pair with a rename on each line can still be flagged.

## Tests

New rows in the tier-0 `TestFindNewWrappedBreaks` table cover a rename of a line's last word and of its first word (not flagged), a new break beside a rename (flagged), and a line pair rewritten in most of its words (flagged).
