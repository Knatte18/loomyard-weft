---
round: 1
ledger:
  - key: remove-refill-failed-delete-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: prune-third-pass-negative-rows-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: pair-worktree-stands-fails-open
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: path-taken-halt-registered-worktree
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: boardcli-plain-dir-seam-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none gating.
Two are MEDIUM test-coverage gaps (F1, F2), two are LOW design edge cases (F3, F4), and one is a LOW coverage gap (F5).
