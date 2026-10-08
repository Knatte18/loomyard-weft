---
round: 2
ledger:
  - key: card5-untagged-seam-spawns-go-list
    rounds: [1]
    status: resolved
  - key: card5-per-identifier-fallback-no-mechanism
    rounds: [1]
    status: resolved
  - key: card4-import-selector-exclusion-unbounded
    rounds: [1]
    status: resolved
  - key: card3-nested-module-test-row-contradicts-rule
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card5-typed-and-scan-merge-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card4-scanreferences-doc-comment-stale
    rounds: [2]
    status: open
    class: scope
    severity: NIT
---

Round 1 raised three findings, led by a BLOCKING design finding on card 5's untagged seam.
Round 2's review raised none of those again, so all three are resolved.
Round 2 raised three new findings, none design-class: a contradictory test row in card 3 (MEDIUM), an unstated merge of typed and scan results in card 5 (LOW) and a stale `scanReferences` doc comment in card 4 (NIT).
