---
round: 1
ledger:
  - key: card1-rubric-stencil-unlicensed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card1-check-tally-sweep-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card1-fixture-reorder-unnamed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: cobra-validate-drives-repeated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card3-spec-mention-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: card4-unknown-role-test-not-deleted
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card6-test-duplicates-existing
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: card8-stuck-strand-claim-wrong
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card8-warning-outcome-wording
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card9-commitsnotin-backend
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: remoteonlycommits-needs-own-card
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card10-checkout-failed-refusal-untested
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card10-failed-checkout-envelope
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card10-stale-docs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: verify-skips-loomcli-tmux
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 is the first review, so every key opens here.
The gating items are the two BLOCKING findings, a test duplicating existing coverage in card 6 and an untested refusal in card 10, plus three MEDIUM design findings in cards 8, 9 and 10.
Cards 1, 3 and 4 carry scope and consistency gaps that do not gate.
