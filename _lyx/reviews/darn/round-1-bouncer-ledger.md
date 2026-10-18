---
round: 1
ledger:
  - key: implementers-second-registry-map
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: implementer-config-passthrough-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: implementer-template-default-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: implementer-setting-nonstring-refusal
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: creel-test-change-unlicensed
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings.
The implementers table conflicts with the single-registry-map invariant and needs a PATTERN and header amendment, or a fold into the one map.
The rest are test gaps, a refusal message without a way forward, and an unlicensed creel test hunk.
