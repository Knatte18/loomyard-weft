---
round: 4
ledger:
  - key: d4-bound-false-for-per-worktree-modules
    rounds: [1, 2, 3, 4]
    status: open
    class: design
    severity: BLOCKING
  - key: seed-only-upper-layers-pruned
    rounds: [1]
    status: resolved
  - key: d6-strip-silently-inherits-equal-values
    rounds: [1, 2, 3]
    status: resolved
  - key: d9-removal-discards-hand-written-local-override
    rounds: [1]
    status: resolved
  - key: d4-rationale-overlooks-env-expansion
    rounds: [2, 3]
    status: resolved
  - key: d6-migration-adds-keys-under-worktree
    rounds: [3]
    status: resolved
  - key: d1-env-expansion-across-layers-undecided
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
  - key: d6-strip-acceptance-misses-reverse-marker
    rounds: [4]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised four design findings; rounds 2 and 3 resolved the seed-only pruning and D9 removal findings.
The D4 bound finding has recurred in every round: per-worktree keys in rounds 1 and 2, hub-wide landing and darn keys in round 3, and `loom.discussion_producer` in round 4.
The shared cause is that `LocalKeys` and `LocalDenied` are enumerated rather than derived from D4's category test.
Round 4 resolved the D4 `.env` rationale, the D6 strip comparison and the D6 migration findings.
It added a MEDIUM decision finding on which `.env` expands a marker once layers stack, and a LOW scope finding that the strip's acceptance row misses the reverse marker case.
