---
round: 2
ledger:
  - key: seed-signature-matches-stencil-sync
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d5-drop-refused-with-board-commit-ahead
    rounds: [1]
    status: resolved
  - key: d1-ancestry-unanswerable-outside-loomyard-and-specs
    rounds: [1]
    status: resolved
  - key: stamp-retry-cannot-clear-hubwide-config-commit
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: ancestry-read-mechanism-contradiction
    rounds: [1]
    status: resolved
  - key: d9-bound-rests-on-false-d8-claim
    rounds: [1]
    status: resolved
  - key: failing-probe-silently-stops-gc
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-miss-behaviours
    rounds: [1]
    status: resolved
  - key: unresolvable-terms-for-plan-producer
    rounds: [1]
    status: resolved
  - key: forced-refresh-nonprod-binary-freezes-copy
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised three BLOCKING design findings on D5's drop signature and primitive and D1's ancestry source, plus consistency and LOW gaps.
Round 2 re-raised none of them at gating weight.
The signature finding survives only as the pre-change-binary transition case (F2, LOW consistency).
The hub-wide config commit finding survives as the narrower claim that a board push cannot clear a conflicting commit (F3, LOW consistency).
The other round-1 keys were not re-raised and are resolved.
Round 2 adds one new design MEDIUM: a forced refresh from a dev or modified binary can freeze the copy against production binaries (F1).
