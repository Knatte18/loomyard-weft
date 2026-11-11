---
round: 1
ledger:
  - key: droplocal-untracked-removal-untested-negatives
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: adopt-reconcile-pushlockwait-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: adopt-doc-step-done-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-adopt-entry-missing-behaviours
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reconcile-write-lock-nonzero-wait-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: adopt-rejudge-still-holding-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all open and all with a fix the review text already fixes.
F1 and F2 are the gating pair: each asks for a test that pins behaviour the diff adds.
F1 covers the negative cases of the destructive untracked-file removal in `DropLocal`.
F2 covers the adopt's push-lock bound passed to its reconcile.
F3 and F4 are doc drift in `internal/adoptcli/doc.go` and `docs/overview.md`.
F5 and F6 are test gaps at LOW: the write lock held under a nonzero wait, and a re-judge of a stop that still holds.
