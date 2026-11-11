---
round: 2
ledger:
  - key: card4-unexported-helper-tests-repeat-name-set
    rounds: [1]
    status: resolved
  - key: card1-competing-write-subtest-redundant
    rounds: [1]
    status: resolved
  - key: card1-takeover-docs-false-with-replace-set
    rounds: [1]
    status: resolved
  - key: card4-unpushed-read-error-no-outcome
    rounds: [1]
    status: resolved
  - key: card4-failed-hub-way-forward-untestable-condition
    rounds: [1]
    status: resolved
  - key: card4-fakehubs-seam-count
    rounds: [1]
    status: resolved
  - key: card2-fromthishost-no-stated-bound
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card4-newnamemovestotheend-row-no-input
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card4-failedhub-realhub-subtest-duplicates-tier0
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1's six findings are absent from round 2's review, which states its fixes hold, so all six are resolved.
Round 2 raises three LOW design findings: an unstated bound on card 2's `FromThisHost`, a card 4 test row with no input that repeats a sibling, and a card 4 tier 1 subtest that repeats tier 0 and `boardengine` coverage.
Each fix is a local correction the finding text fixes.
