---
round: 1
ledger:
  - key: sibling-card-cross-references-wrong
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: driver-silence-hidden-effect-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: notify-rule-escapes-recipe-blindness-scan
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card1-replans-existing-compact-tests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card5-arm-tests-not-separated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: cards7-8-tests-beside-existing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: unused-uses-entries
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised seven findings (F1 to F7 in order of the keys above), one gating.
The gating one is the unstated bound on the driver silence in cards 3 and 6.
The rest are wrong sibling-card references, the recipe-blindness scan gap for the moved notify stencils, three test-economy overlaps and one stray Uses entry pair.
