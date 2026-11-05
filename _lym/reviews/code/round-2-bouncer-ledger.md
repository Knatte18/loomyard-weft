---
round: 2
ledger:
  - key: amend-route-dead-ends-in-exhausted-recover-batch
    rounds: [1]
    status: resolved
  - key: create-way-forward-arms-and-precedence-untested
    rounds: [1]
    status: resolved
  - key: rename-new-side-told-to-amend-create-target
    rounds: [1]
    status: resolved
  - key: signature-mismatches-no-negative-rows
    rounds: [1]
    status: resolved
  - key: detail-compared-with-contains
    rounds: [1]
    status: resolved
  - key: rename-not-done-flag-missing
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: exhausted-ignores-pending-amendment
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

All five round-1 findings were not raised again in round 2 and are carried as resolved.
Round 2 raised two new findings.
F1: a Rename new side answered `not_found` is check `rename-not-done`, which `cardNotDoneFor` does not flag, so it keeps the default way forward.
F2: `failCardNotDone` decides exhaustion by `Recoveries` alone, though `RecoverSpawnOrAttach` also weighs an unrendered amendment.
Both fixes are local and spelled out in the findings.
