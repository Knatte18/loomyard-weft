---
round: 2
ledger:
  - key: card1-rubric-stencil-unlicensed
    rounds: [1]
    status: resolved
  - key: card1-check-tally-sweep-incomplete
    rounds: [1]
    status: resolved
  - key: card1-fixture-reorder-unnamed
    rounds: [1]
    status: resolved
  - key: cobra-validate-drives-repeated
    rounds: [1]
    status: resolved
  - key: card3-spec-mention-open
    rounds: [1]
    status: resolved
  - key: card4-unknown-role-test-not-deleted
    rounds: [1]
    status: resolved
  - key: card6-test-duplicates-existing
    rounds: [1]
    status: resolved
  - key: card8-stuck-strand-claim-wrong
    rounds: [1]
    status: resolved
  - key: card8-warning-outcome-wording
    rounds: [1]
    status: resolved
  - key: card9-commitsnotin-backend
    rounds: [1]
    status: resolved
  - key: remoteonlycommits-needs-own-card
    rounds: [1]
    status: resolved
  - key: card10-checkout-failed-refusal-untested
    rounds: [1]
    status: resolved
  - key: card10-failed-checkout-envelope
    rounds: [1]
    status: resolved
  - key: card10-stale-docs
    rounds: [1]
    status: resolved
  - key: verify-skips-loomcli-tmux
    rounds: [1]
    status: resolved
  - key: leased-update-no-lease-lost-sentinel
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: gitrepo-boundary-test-method-set
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: remote-force-update-no-gated-executor
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card1-runlevel-backward-uses-tests
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card8-paused-outcome-omitted
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card6-done-row-repeats-tests
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card11-reset-rows-repeat-tests
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card3-whole-plan-row-repeats-scope-test
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: cards2-3-check-ids-no-spec-row
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

None of the round-1 keys reappears in round 2's fresh review, so all fifteen are resolved.
Round 2 raises nine new findings, all in the reworked text.
The gate is F1: card 9's leased update gives reset no way to tell a lost lease from another update failure, though the two need different refusals.
F2 and F3 are the gitrepo boundary test and the destructive-guard test, which pin method and executor sets no card edits.
F4 is two `Run` tests that build a plan the new check refuses.
F5 to F9 are LOW test-economy and documentation gaps.
