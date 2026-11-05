---
round: 1
ledger:
  - key: judge-snapshot-counts-loom-status-commits
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: duplicate-delete-verifyfixture-helpers
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: cards-4-16-26-miss-test-members
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: drift-amend-refusal-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: check-before-expect-names-defer
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: checkbaseline-unguarded-hides-missing-dir
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card-18-names-removed-verbs-in-headers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: capture-lock-wait-no-cadence-no-pattern-listing
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stale-docs-drift-rows-and-verbs
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: defer-held-lock-wrong-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: countedcapture-left-without-caller
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so the ledger holds only this round's findings, all open.
Four are BLOCKING design findings: the judge snapshot treating loom's own status commits as judge edits, the duplicate helper deletes between cards 16 and 25, the test members cards 4, 16 and 26 miss, and the `lym drift amend` refusal with no way forward.
Five MEDIUM and two LOW findings cover consistency and scope gaps.
Only the judge snapshot fix introduces a new seam, so it is the one marked `design-wrong`.
