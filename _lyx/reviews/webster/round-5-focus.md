---
round: 5
focus:
  - "Sweep every `//` line that `git log --first-parent --no-merges -p` or the merge-base diff adds or changes in `internal/fabricengine`, `internal/fabriccli`, `internal/landingshed`, `internal/loomcli` and `internal/hubgeom`, test files included, and settle for each multi-clause sentence whether every independent clause (a comma plus `so`, `and`, `but` or `or` followed by its own subject and verb) starts its own line, as `PATTERN-comment-line-breaks` requires."
  - "Check the same sweep for the opposite defect: a break inside one clause or inside one compound predicate (two predicates sharing a subject), which `PATTERN-comment-line-breaks` forbids."
  - "In `internal/fabricengine/checkout.go` (the `switchOrForkWeft` doc), `internal/fabricengine/remoteleftover.go` (the file header and the `leftoverKind` doc) and `internal/landingshed/deps.go` (the `ConfigChanges` and `Notify` field docs), settle whether the current text satisfies that rule line by line."
---

The three entries name where to look and the question to settle.
The sites in the third entry are the files whose comments were most recently rewritten, so they are the likeliest to hold a new defect from the rewrite itself.
