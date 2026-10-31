---
round: 1
ledger:
  - key: missing-upstream-judged-divergence
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: fetch-failure-branch-untested
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: sentinel-new-text-not-pinned
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: plain-retry-way-forward-not-asserted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unwrapped-claim-in-header-and-test-name
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none seen before.
F1 (`missing-upstream-judged-divergence`) is the only gating one: `failRejection` can record `not_fast_forward` when `HasUnpulled` returns its conservative true for a missing upstream.
F2 to F5 are test-coverage and wording gaps in the new rows, the sentinel text and a stale header.
