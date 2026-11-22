---
round: 1
ledger:
  - key: fresh-run-seed-child-still-pushes
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: failed-start-push-not-retried-on-resume
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: start-branch-uncovered-behaviours
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: resumed-start-row-narrowing-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: pair-remote-entry-lacks-start-row-exception
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: statuscommit-push-contract-contradicted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seam-doc-says-start-commits-nothing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, mapped in order F1 to F7.
F1 (BLOCKING): `seedAbsentStatus` seeds Worktree-Create without a seam call, so the first seam call is Seed-Child and pushes.
F2 and F3 (MEDIUM design): a failed or interrupted start push is not retried on resume, and four start-branch behaviours have no covering row.
F4 to F7 are LOW: an unrecorded narrowing deferred to the parent, a PATTERN-pair-remote headline gap, a `statuscommit` doc contradiction and a seam doc wording slip.
