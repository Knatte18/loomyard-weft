---
round: 4
focus:
  - "Every Go comment line that the branch adds or changes relative to `git merge-base main HEAD`, in all packages touched (yamlengine, configengine, configcli, boardengine, boardcli, including `doc.go` files and test files): does any line hold two independent clauses, joined by a semicolon or by a comma plus `and`, `but`, `or` or `so`, or hold two sentences? Check every changed comment line in the diff, not a sample, against `PATTERN-comment-line-breaks`."
  - "Comments recently re-broken to fix this rule: did the fix itself leave a line with two clauses, or a break in the middle of a clause? Re-read each re-broken comment block in full."
---

The directive stays the one rule, `PATTERN-comment-line-breaks`.
A line that ends at its semicolon, with the next clause on the following line, complies.
Compound predicates and subordinate `which`/`where`/`that` clauses are not violations.
