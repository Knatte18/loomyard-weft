---
round: 1
ledger:
  - key: card3-drift-testenv-seams-unlisted
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card10-routing-progressat-pin-unlisted
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card7-review-input-note-fixed-at-build
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card8-findings-file-location-dirty-tree
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card7-shed-recipe-spec-reviewinput
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card10-prreview-stuck-always-halt-docs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card9-prreview-type-doc-unlisted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-trailer-escape-hatch-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, none recurring since there is no earlier ledger.
Two BLOCKING scope gaps are tests or kits a card breaks without listing them (cards 3 and 10).
Two MEDIUM design findings need fresh review of their fixes: where Code-Burler-Review gets its review-input note after a reroute in one process (card 7), and where the PR-Review agent writes its findings file (cards 8 and 12).
The rest are unlisted docs and specs, plus one unstated bound on card 3's trailer escape hatch.
