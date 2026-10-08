---
round: 1
ledger:
  - key: prune-collides-with-locked-field-rule
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: batch-merge-comparison-base-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: abandoned-status-locked-by-predicate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: failed-pair-creation-leaves-dangling-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: unlock-of-live-run-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: done-or-absent-entry-never-locked
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: decision-6-heading-overstates-ordering
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: rejected-alternative-trail-in-rationales
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 (F1 to F8) raised eight findings, all open.
The gating ones are the prune collision (F1), the `abandoned` status caught by the lock predicate (F3) and the dangling lock after a failed pair creation (F4).
