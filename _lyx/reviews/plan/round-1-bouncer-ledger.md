---
round: 1
ledger:
  - key: card27-traced-glyph-regex
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card11-loomshed-yaml-allowlist
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card25-treesitter-link-guard
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card4-removes-prescribed-counts-done-cards
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: round-start-whole-second
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card4-line-unprescribed-frozen-card
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card13-darn-refusal-order
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-overviewat-empty-revision-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card11-undecodable-record-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card12-unparsable-round-start
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: card13-plan-parse-failure-retry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card6-stale-seven-label-lists
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card6-planparser-doc-deferred-to-card8
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card13-amends-plan-shadows-no-record
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: card18-amendmenttarget-bool-signature
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: design-wrong
---

Round 1 raised 15 findings, all open.
F1 (BLOCKING, design) and F4 (MEDIUM, design) gate the round.
F2 and F3 are MEDIUM scope findings about guard lists that cards 11 and 25 must extend.
The rest are LOW or NIT way-forward, reachability and consistency gaps.
Keys marked `design-wrong` change a signature or behavior, so a fresh reviewer must see the new text.
