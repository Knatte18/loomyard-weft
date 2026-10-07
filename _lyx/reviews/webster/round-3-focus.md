---
round: 3
focus:
  - "Comment line breaks in the diff `git diff $(git merge-base main HEAD)..HEAD -- '*.go'`: for every added or changed `//` comment sentence, does each line break fall at a sentence end or at an independent-clause boundary (a semicolon, or a comma before a coordinating conjunction that opens a clause with its own subject and verb)? Check that no break falls inside a clause, for example before a fragment that follows a colon or comma, or between the predicates of a compound predicate. Start with `internal/fabricengine/checkout.go`, `mutation.go`, `reconcile.go` and `internal/landingshed/finalize.go`."
  - "In the same diff, does any added or changed comment line hold two independent clauses joined by a semicolon, or by a comma plus `so`, `and` or `but`, with no break between them? Sweep added `//` lines containing `; ` or `, so `/`, and `/`, but `, judge each against the independent-clause test, and leave untouched legacy lines, short compound-predicate or list sentences, and semicolons inside parentheses alone."
  - "`internal/fabricengine/remoteleftover.go`: does the `ErrRemoteLeftover` doc keep the list that follows its colon on the colon's own line, rather than splitting the fragments onto the next line?"
---

Every entry applies `PATTERN-comment-line-breaks` to the committed diff only.
