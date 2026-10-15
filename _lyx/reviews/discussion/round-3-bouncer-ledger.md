---
round: 3
ledger:
  - key: stale-local-ttl-key-refuses-shuttle-load
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: retired-key-open-map-flag-no-consumer
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-risks-omits-python-switch
    rounds: [1]
    status: resolved
  - key: sparse-layer-template-default-overwrite
    rounds: [1]
    status: resolved
  - key: acceptance-row-deny-python-run-settings
    rounds: [1]
    status: resolved
  - key: d8-compile-error-claim-vs-empty-union
    rounds: [1]
    status: resolved
  - key: d5-rationale-rejected-alternative-narrative
    rounds: [1]
    status: resolved
---

Round 1 raised six findings, one BLOCKING.
Round 2 reopened the stale-TTL key at MEDIUM over an unspecified mechanism.
Round 3 maps F2 to that key at LOW: the mechanism is now stated, and only the shape bound for a retired key is left open.
F1 is a new LOW consistency key about an open-map flag in D4 that no rule consumes.
The five other round 1 keys stay resolved.
