---
round: 1
ledger:
  - key: audit-replaces-equal-entries
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: dobby-exit-record-unread
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: dobby-batch-every-keys-live
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: miss-with-entries-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: redundancy-cover-rules-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: outcome-entries-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: self-check-linked-funcs-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: removed-test-keeps-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-failure-survives-restamp
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: self-check-skipped-without-profile
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: legacy-entries-age-from-builtat
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-refresh-wording-dobbyengine
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: audit-guards-and-verb-output-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: plan-fallbacks-and-no-slot-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review; the ledger holds its fourteen findings, F1 to F14 in order.
Three are `decision` findings deferred to the parent: the audit's equal-result replacement, the unread exit record, and the live dobby keys.
Seven test-coverage and code-correctness `design` findings at MEDIUM or LOW gate the loop, plus one LOW wording `consistency` finding.
