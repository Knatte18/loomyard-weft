---
round: 1
ledger:
  - key: lift-full-pass-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: draft-migration-after-mutation
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-lift-pending
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: arbiter-stop-comment-source
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings.
F1 maps to `lift-full-pass-unreachable`, F2 to `draft-migration-after-mutation`, F3 to `stale-lift-pending` and F4 to `arbiter-stop-comment-source`.
F2 is marked `design-wrong` because its fix offers two approaches and the chosen one needs a fresh review.
