---
round: 2
focus:
  - "decision-record.md D2: the guard-directory drop clause. Settle whether a guard package directory drops out of the guard step only for the renaming or deleting card and later cards of the batch, and only when every Go file of that directory is renamed out or deleted, so a directory still holding Go files stays in the step. Check the Bound states what the drop skips and what still catches those tests, and that the Acceptance criteria row for the drop agrees with the clause."
  - "decision-record.md D3: a guard scan error degrades the guard step for the batch. Settle whether the Bound says how long the error persists, given the scan reads the tree and a bad file recurs at every later begin-batch, and whether a walk read error is justified by the Rationale or has its own way forward. Check Open risks covers it."
  - "decision-record.md D4: the fork may edit a guard test to make the guard step pass. Settle whether the Bound states what this lets through (a loosened pinned set or allowlist), what steers the fork to the code fix first, and what keeps a wrong loosening visible. Check Open risks covers it."
  - "decision-record.md D2 directory walk: settle whether the set of directories the walk scans matches what `go list ./...` treats as packages, in particular `testdata` and directories whose names begin with `_` or `.`, and that an acceptance row covers it."
---

Targets the design findings of round 1; the consistency and NIT findings are mechanical text fixes.
