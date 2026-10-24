---
round: 3
ledger:
  - key: card-23-pattern-shorten-order
    rounds: [1]
    status: resolved
  - key: hand-enumerated-affected-sets
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: retag-cards-custom-not-edit
    rounds: [1]
    status: resolved
  - key: missing-resign-arrows-darntreescope-guardrunof
    rounds: [1]
    status: resolved
  - key: darnrouting-cannot-assert-entry-gate
    rounds: [1]
    status: resolved
  - key: entry-gate-error-points-at-run-settings
    rounds: [1]
    status: resolved
  - key: entry-gate-error-generic-resume
    rounds: [1]
    status: resolved
  - key: bare-config-reconcile-way-forward
    rounds: [1]
    status: resolved
  - key: card-24-verify-empty-covers
    rounds: [1]
    status: resolved
  - key: d13-follow-up-underspecified
    rounds: [1]
    status: resolved
  - key: default-gatekey-branch-test-only
    rounds: [1]
    status: resolved
  - key: greenrecord-duplicates-scenario-rows
    rounds: [1]
    status: resolved
  - key: retag-support-files-old-tag-spellings
    rounds: [2]
    status: resolved
  - key: card-20-duplicate-forced-answer-test
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card-21-extends-should-be-edit
    rounds: [2]
    status: resolved
  - key: card-15-extends-should-be-edit
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card-17-extends-should-be-edit
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: prose-naming-gate-flags-and-old-tags
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1's findings were resolved except the affected-set method gap, which recurs in every round.
Round 3 repeats it a third time (BLOCKING, design) on cards 14, 20 and 13, so the verdict is CIRCLING.
Round 2's retag-support-files key is resolved: round 3's review confirms every test-holding directory sits in a retag card.
Round 2's card 21 Extends typing is resolved on silence; round 3 raises the same defect shape on cards 15 and 17 under new keys.
Card 20's forced-answer redundancy reopens in round 3 with a narrower scope.
Round 3's LOW scope finding covers prose in `docs/overview.md` and `crucible/` that names the gate flags and old tags.
