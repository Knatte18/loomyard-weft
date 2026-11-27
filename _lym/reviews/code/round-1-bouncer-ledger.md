---
round: 1
ledger:
  - key: transient-failure-persisted-as-failed
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: transient-stop-texts-overstate-network-and-redrive
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: parent-lock-wait-suspend-claim-too-strong
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: build-skew-over-transient-precedence-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 opened every key; there is no earlier ledger.

- `transient-failure-persisted-as-failed` (F1): the amended entry asks for a retryable stop in place of `failed`,
  and the diff changes only the halt note's anomaly kind.
  The fix changes the persisted state and its readers, unless the run's parent rules the narrower reading in scope.
- `transient-stop-texts-overstate-network-and-redrive` (F2): three texts describe every transient-stop halt
  as a network or host failure that the driver re-steps.
- `parent-lock-wait-suspend-claim-too-strong` (F3): the `ensure` doc comment states a host suspend as fact
  from a gap between wall-clock and awake time.
- `build-skew-over-transient-precedence-untested` (F4): no `TestClassifyHalt` row sets both inputs.
