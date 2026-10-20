---
round: 1
ledger:
  - key: old-binary-reseed-reverts-fans
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: template-edits-stop-reaching-hub
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: template-lens-fan-cannot-be-deleted
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: set-on-any-name-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: hub-only-entries-warned-unread
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: expansion-bound-wrong-exposure
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: deletion-staging-already-settled
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: missing-acceptance-criteria
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: kept-target-line-renders-whole-mappings
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings (F1 to F9), in this key order.
The first five are the substantive ones: an old binary's re-seeded `burler.yaml` can revert a migrated fan, template edits stop reaching a hub, template entries cannot be deleted, `--set` on any name has no bound, and hub-only entries trip the unknown-keys warning.
The last four are local corrections: the expansion bound's wording, an already-settled deletion staging, two missing acceptance criteria, and the kept-target line's rendering.
