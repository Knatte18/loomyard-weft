---
round: 1
ledger:
  - key: fabric-add-bypasses-spawnpairpush
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: fabric-add-envelope-and-spawnpairpush-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: batten-create-and-seed-push-spawns-unasserted
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: destroy-stale-add-step-number
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: addwarpbranch-one-field-wrapper
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so no prior ledger exists.
The review raised five findings: three MEDIUM, one LOW, one NIT.
None is gating: the only `design` finding is a NIT, and no finding is BLOCKING.
