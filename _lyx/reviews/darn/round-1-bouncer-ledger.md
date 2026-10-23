---
round: 1
ledger:
  - key: readme-prime-owner-line-unpinned-at-render-sites
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: board-primeowner-absent-directory-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: find-help-claims-list-json
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, none carried from an earlier round.
The MEDIUM design finding is a test gap: no assertion on the rendered README's prime-owner line at the write, owner-set and replay render sites.
The two LOW findings are an uncovered absent-board branch of `Board.PrimeOwner` and a stale `find` help sentence.
