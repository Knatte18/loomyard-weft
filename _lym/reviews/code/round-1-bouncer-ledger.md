---
round: 1
ledger:
  - key: notice-receiver-resolution-unchanged
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: notice-file-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: wake-no-owned-kept-entry-contradictory-notice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: wake-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all class `design`.
F1 maps to `notice-receiver-resolution-unchanged`: the wake notice's receiver is still resolved by `NotifyPrime`, only a retry was added, and a dropped notice for want of a recorded orch is silent.
F2 maps to `notice-file-branches-untested`: the multi-line short-line routing and the notice-file write-failure fallback have no `TestWatch` row.
F3 maps to `wake-no-owned-kept-entry-contradictory-notice`: with no `Owned` and a kept entry the reconcile is skipped and the notice contradicts itself.
F4 maps to `wake-failure-paths-untested`: the done-phase write failure after a lift and the long-reason fail notice are untested.
