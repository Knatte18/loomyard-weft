---
round: 1
ledger:
  - key: card4-card-packs-column-contradicts-test-row
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: card4-card-arm-spans-biased-window
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card2-standalone-runs-lose-pack-default-on
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: card2-empty-card-test-row-conflates-cases
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card2-stale-packcard-doc-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
F1 (card 4's card packs column rule yields `0/0` where its test row wants `-`) is the only BLOCKING one.
F2 (card-arm spans measured against the first edit, biasing the layout comparison) is the gating design finding.
F3 (standalone webster runs lose the pack under the default-on per-card layout) needs an author call or a decision-record amendment.
F4 and F5 are local test-row and doc-comment corrections on card 2.
