---
round: 2
ledger:
  - key: prune-collides-with-locked-field-rule
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: batch-merge-comparison-base-unstated
    rounds: [1]
    status: resolved
  - key: abandoned-status-locked-by-predicate
    rounds: [1]
    status: resolved
  - key: failed-pair-creation-leaves-dangling-lock
    rounds: [1]
    status: resolved
  - key: unlock-of-live-run-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: done-or-absent-entry-never-locked
    rounds: [1]
    status: resolved
  - key: decision-6-heading-overstates-ordering
    rounds: [1]
    status: resolved
  - key: rejected-alternative-trail-in-rationales
    rounds: [1]
    status: resolved
---

Round 1 raised eight findings; round 2 raised two, and six round-1 keys did not recur.

Two keys stay open.
`prune-collides-with-locked-field-rule` (F1): Decision 4's enforcement rule has no exception for the prune that Decision 2 and an Acceptance criterion require.
`unlock-of-live-run-unbounded` (F2): the bound is a `lyx batten status` check that refuses outside the prime and whose output has no stated mapping to no run in progress.
