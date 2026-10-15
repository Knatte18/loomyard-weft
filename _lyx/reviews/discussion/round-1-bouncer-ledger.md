---
round: 1
ledger:
  - key: stale-local-ttl-key-refuses-shuttle-load
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: open-risks-omits-python-switch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: sparse-layer-template-default-overwrite
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: acceptance-row-deny-python-run-settings
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-compile-error-claim-vs-empty-union
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: d5-rationale-rejected-alternative-narrative
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings.
The gating one is F1: unreconciled local-layer TTL keys make shuttle's loader refuse, which breaks D2's stop and teardown guarantee.
F3 and F5 change decisions (D4's overwrite scope, D8's empty-set guard) and need fresh review.
F2, F4 and F6 are local text corrections.
