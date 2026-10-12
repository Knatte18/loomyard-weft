---
round: 2
ledger:
  - key: plan-severity-rule-marker-slot-fit
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: fabric-vocabulary-criteria-unreached-branches
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: plan-producer-default-rationale-false-premise
    rounds: [1]
    status: resolved
  - key: fork-give-up-risk-overstates-landing
    rounds: [1]
    status: resolved
  - key: redefinable-fan-bound-unstated
    rounds: [1]
    status: resolved
  - key: verbatim-host-texts-only-in-support-log
    rounds: [1]
    status: resolved
  - key: fork-give-up-risk-stated-twice
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

The Plan severity rule defect survives round 1's fix.
The fix applied the proposed value, but its trailing evidence clause still breaks D4's noun-phrase slot, which makes the finding BLOCKING in round 2.
The fabric-vocabulary criteria still miss one branch, a side token in a method's type segment alone.
The fork give-up risk now appears in both D2 and Open risks, which is a duplication, not a wrong figure.
The other four round-1 findings were checked against the code and the record and are resolved.
