# Decision record — comment-lint-rename

## Goal

The comment lint in `internal/commentlint` flags no break of a legacy wrapped comment paragraph whose lines an edit changed only by renaming words in place.

## Scope

- In: the new-break test in `internal/commentlint`, its tier-0 table rows and the package doc in `internal/commentlint/doc.go`.
- Out: the semantic-break rules (sentence end, semicolon, comma before a clause), the skipped paragraph kinds, the diff and file reading in `diff.go`, and the gate that calls the lint.

## Decisions

### A break is carried over when its two lines are a base line pair with words substituted in place

Decision: a break that the base-pair lookup calls new is still not new when the base text holds two consecutive prose lines of one paragraph with the same word counts as the break's two lines, whose words equal the break's words position by position except for substituted words, and whose unchanged words outnumber the substituted ones across the two lines.
Rationale: a word rename keeps every line end where it was and changes only words in place, so each line keeps its word count and most of its words;
the word pair across a break changes only when the renamed word sits at the break.

Bound: the rule lets through a break whose line pair matches a base line pair word for word apart from a minority of in-place substitutions.
A reflow that moves a word across a line end changes both lines' word counts, so it never matches the pair it came from;
it matches another base pair only when that pair has the new word counts and agrees with the new lines on most positions.
A rename that changes a word count, such as one word replaced by two, is not covered and still flags the break.

### The match runs only when the existing checks flag the break

Decision: the substitution match runs after the word-pair lookup and the semantic-break check have both failed, over base line pairs indexed by their two word counts.
Rationale: most breaks are settled by the cheap checks, and the index keeps the match to pairs that can match at all.

## Constraints

- `PATTERN-test-speed`: the new cases are rows in the existing tier-0 `TestFindNewWrappedBreaks` table.
- `PATTERN-gitrepo-client-boundary` is untouched: no new git call.

## Auto-mode assumptions

- The rename in the reported halt replaced one token by one token (a binary name), so a same-word-count rule covers it.
- A majority of unchanged words is the threshold; a wrapped prose line pair holds far more words than a rename touches.

## Open risks

- A short line pair, such as two lines of two words each with a rename on each line, has no unchanged majority and is still flagged.

## Acceptance criteria

- A legacy wrapped paragraph whose line-end word is renamed is not flagged.
- A legacy wrapped paragraph whose line-start word is renamed is not flagged.
- A break an edit creates in a legacy wrapped paragraph that also carries a rename is still flagged.
- A line pair rewritten in most of its words is flagged.
- Every existing row of `TestFindNewWrappedBreaks` keeps its result.
