---
round: 4
focus:
  - "Sweep every Go comment line the range `git diff $(git merge-base main HEAD)..HEAD` adds or changes, in all touched packages (`internal/fabricengine`, `internal/fabriccli`, `internal/landingshed`, `internal/loomcli`), test files included. Settle for each line whether a break falls inside a clause, for example after a comma inside a list that follows a colon, or between the two predicates of one compound predicate, leaving a fragment with no verb of its own on the next line."
  - "In the same sweep, settle for each added or changed comment line whether it holds two or more independent clauses, each with its own subject and verb, joined by a semicolon or by a comma plus `and`, `or`, `but` or `so`, with no line break between them. Apply `PATTERN-comment-line-breaks`: a compound predicate or a list stays on one line, and moved legacy lines that were only re-indented are not changed text."
  - "Check the text that earlier fixes in the range introduced, such as the header comment of `internal/fabricengine/remoteleftover.go`, `internal/fabricengine/add.go` near the live-pair list, `internal/fabricengine/configchanges.go` and the `doc.go` section on weft branch reconciliation, against both rules above, since a re-broken line is new text."
---

Each entry names the whole range because a per-site list has left sites behind.
