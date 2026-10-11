---
round: 3
ledger:
  - key: d4-bound-false-for-per-worktree-modules
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: seed-only-upper-layers-pruned
    rounds: [1]
    status: resolved
  - key: d6-strip-silently-inherits-equal-values
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-removal-discards-hand-written-local-override
    rounds: [1]
    status: resolved
  - key: d4-rationale-overlooks-env-expansion
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-migration-adds-keys-under-worktree
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 1 raised four design findings; round 2 resolved the seed-only pruning and D9 removal findings.
The D4 bound finding has recurred in every round: per-worktree keys in rounds 1 and 2, hub-wide landing and darn keys in round 3.
The shared cause is that `LocalKeys` is enumerated rather than derived from D4's category test.
The D6 strip finding recurs for the third round, now over how `${env:...}` markers are compared.
The D4 rationale finding rose from LOW to MEDIUM because the per-worktree `.env` is an untracked route to every marker-carrying denied key.
Round 3 adds one LOW finding: migration adds keys beneath a worktree whose file lacks them, contradicting the "resolves the same" claim.
