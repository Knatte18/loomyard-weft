---
round: 1
ledger:
  - key: gate-slot-row-edit-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card3-unexported-helper-tests
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: card11-darnrecipe-test-redundant
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: card1-verify-seams-missed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card6-whole-diff-empty-command-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card6-tests-miss-whole-diff-behaviors
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card3-claim-clear-unstated-effect
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: darn-recipe-arm-skip-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: dirtypaths-wider-than-tracked
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card13-params-test-nonexistent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: verified-tree-background-stale-opening
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eleven findings, none seen before.
Gating: the gate-slot row edit no card carries (F1), the two test-economy violations on cards 3 and 11 (F2, F3), and three design gaps in attack-surface statements on cards 6, 3 and 12-13 (F5, F7, F8).
Non-gating: card 1's missed webster verify seams (F4), card 6's missing whole-diff tests (F6), and three LOW consistency items (F9-F11).
