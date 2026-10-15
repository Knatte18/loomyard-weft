---
round: 2
ledger:
  - key: stale-local-ttl-key-refuses-shuttle-load
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
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

Round 1 raised six findings, one BLOCKING and design-class.
Round 2 raised one: F1, a MEDIUM design finding that D4's retired-key rule has no stated home for the category and open-map shape.
It reopens `stale-local-ttl-key-refuses-shuttle-load` at lower severity, because the BLOCKING failure is mostly closed but the mechanism is unspecified.
The other five round 1 keys were not raised again, and round 2's Rejected section confirms the sparse-layer and D8 fixes, so they are resolved.
