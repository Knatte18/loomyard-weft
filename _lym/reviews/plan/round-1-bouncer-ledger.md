---
round: 1
ledger:
  - key: beam-pr-mode-reads-github-rules-unconditionally
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: second-retired-row-mechanism-unreconciled
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: old-flow-text-survives-every-card
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: empty-beam-required-parent-no-way-to-ready
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: judge-awaiting-run-failed-halt-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: accept-removal-order-crash-window
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: await-test-rows-unreachable-pr-review-hold
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: required-checks-check-runs-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised eight findings: F1 to F8 map in order to the keys above.
F1 is the only BLOCKING one: `lym beam ready` cannot pass its pull-request-mode refusal on a hub with no GitHub origin or token.
F4 is its sibling on the same refusal, where a beam with no commits is sent to a verb that refuses it.
F2 and F3 are doc and naming reconciliation; F5 and F7 are test-row corrections; F6 and F8 are narrower design gaps.
No finding has appeared in an earlier round.
