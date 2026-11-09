---
round: 1
ledger:
  - key: showfile-test-missing-shortsha-tag-dir-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: headcommit-test-missing-symlinked-worktree-row
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no prior ledger.
F1 maps to the `showFile` test-row key and F2 to the `headCommit` symlink-row key.
Both are LOW test-coverage gaps with fixes spelled out in the review.
