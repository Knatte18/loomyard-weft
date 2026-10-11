---
round: 1
ledger:
  - key: d4-bound-false-for-per-worktree-modules
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: seed-only-upper-layers-pruned
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d6-strip-silently-inherits-equal-values
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-removal-discards-hand-written-local-override
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 raised four design findings (F1 to F4), none seen before.
F1 and F2 are BLOCKING: the local-layer bound is false for per-worktree modules, and pruning seed-only upper layers deletes operator-owned keys.
F3 is MEDIUM: the one-time strip's effect on values equal to the seeded default is unstated.
F4 is LOW: the D9 removal's discard window is unnamed.
