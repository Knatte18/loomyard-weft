---
round: 2
ledger:
  - key: exit-record-loses-unmeasured-jobs-on-error
    rounds: [1]
    status: resolved
  - key: no-test-single-slot-hub-pass
    rounds: [1]
    status: resolved
  - key: retake-after-yield-unpinned
    rounds: [1]
    status: resolved
  - key: no-test-head-read-on-slot-wait-wake
    rounds: [1]
    status: resolved
  - key: no-test-named-call-waits-on-build-lock
    rounds: [1]
    status: resolved
  - key: empty-planned-head-on-early-refusal
    rounds: [1]
    status: resolved
  - key: no-test-inherited-slot-call
    rounds: [1]
    status: resolved
  - key: no-test-signal-keeps-finished-rounds
    rounds: [1]
    status: resolved
  - key: no-test-upkeep-cap-default
    rounds: [1]
    status: resolved
  - key: no-test-failed-upkeep-registration
    rounds: [1]
    status: resolved
  - key: stale-exit-file-name-comment
    rounds: [1]
    status: resolved
  - key: audit-round-never-reads-head-before-package-job
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: register-upkeep-pass-not-atomic
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resumed-package-reported-twice
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings; round 2 did not raise any of them again, so all are resolved.
Round 2 raised three LOW findings on new keys: the audit round does not read HEAD before each audit job (F1), `registerUpkeepPass` lists and registers without a lock (F2), and a resumed package enters the built list and measures twice (F3).
Each fix is a local correction its finding text already fixes.
