---
round: 2
ledger:
  - key: audit-replaces-equal-entries
    rounds: [1]
    status: resolved
  - key: dobby-exit-record-unread
    rounds: [1]
    status: resolved
  - key: dobby-batch-every-keys-live
    rounds: [1]
    status: resolved
  - key: miss-with-entries-untested
    rounds: [1]
    status: resolved
  - key: redundancy-cover-rules-untested
    rounds: [1]
    status: resolved
  - key: outcome-entries-unpinned
    rounds: [1]
    status: resolved
  - key: self-check-linked-funcs-unpinned
    rounds: [1]
    status: resolved
  - key: removed-test-keeps-entry
    rounds: [1]
    status: resolved
  - key: stale-failure-survives-restamp
    rounds: [1]
    status: resolved
  - key: self-check-skipped-without-profile
    rounds: [1]
    status: resolved
  - key: legacy-entries-age-from-builtat
    rounds: [1]
    status: resolved
  - key: stale-refresh-wording-dobbyengine
    rounds: [1]
    status: resolved
  - key: audit-guards-and-verb-output-unasserted
    rounds: [1]
    status: resolved
  - key: plan-fallbacks-and-no-slot-unpinned
    rounds: [1]
    status: resolved
  - key: package-doc-undated-entry-rule
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: audit-undated-order-untested
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: build-cancel-untested
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

All fourteen round-1 keys are resolved: round 2's review checked each focus area and rejected a finding on every one.
Round 2 adds three LOW findings: F1 the package doc's stale rule for an undated entry, F2 the missing ordering test for it, and F3 the missing build-cancel test.
None gates the loop.
