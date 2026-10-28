# Support log — comment-lint-rename

## Interview

A darn run holds no interview; the scope is read from the board entry and the code of `internal/commentlint`.

## Rejected alternatives

- Treat a break as known when only one of its two words matches a base break: a common word like `the` ends many base lines, so any new break after it would pass.
- Derive a file-wide rename map from the base and new text and compare word pairs modulo it: it needs a line alignment the lint does not have, and the in-place substitution match gives the same result for a one-token rename.
- Ignore a break whose two lines are both unchanged apart from any edit: without a threshold, any rewrite that keeps word counts would pass.

## Review rounds

_No rounds yet._

## Question ledger

- Self-pick: the substitution threshold is a strict majority of unchanged words across the two lines. Resolved.
- Self-pick: a rename that changes a line's word count stays out of scope. Resolved.
