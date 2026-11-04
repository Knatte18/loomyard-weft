---
round: 1
ledger:
  - key: held-turn-end-redelivery-counts-as-delivered
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: prompt-delivery-guards-and-failure-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: attach-delivery-ignores-covering-pause
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failed-open-leaves-session-directory
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unlicensed-fabricengine-lock-test-change
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: attach-test-rows-restate-run-test-rows
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1, first ledger.
F1 is the gating core: a held-turn-end redelivery reads the operator's turn start as the prompt having gone out, so the prompt is never typed yet is reported delivered.
F2 is the paired coverage gap for the new delivery guards and failure branches.
F3 to F6 are local corrections: a missing covering-pause check on attach, a leftover session directory on a failed open, an unlicensed fabricengine test change, and restated attach test rows.
