---
round: 2
ledger:
  - key: gate-slot-row-edit-uncarried
    rounds: [1]
    status: resolved
  - key: card3-unexported-helper-tests
    rounds: [1]
    status: resolved
  - key: card11-darnrecipe-test-redundant
    rounds: [1]
    status: resolved
  - key: card1-verify-seams-missed
    rounds: [1]
    status: resolved
  - key: card6-whole-diff-empty-command-unbounded
    rounds: [1]
    status: resolved
  - key: card6-tests-miss-whole-diff-behaviors
    rounds: [1]
    status: resolved
  - key: card3-claim-clear-unstated-effect
    rounds: [1]
    status: resolved
  - key: darn-recipe-arm-skip-unbounded
    rounds: [1]
    status: resolved
  - key: dirtypaths-wider-than-tracked
    rounds: [1]
    status: resolved
  - key: card13-params-test-nonexistent
    rounds: [1]
    status: resolved
  - key: verified-tree-background-stale-opening
    rounds: [1]
    status: resolved
  - key: card3-gone-worktree-claim-unsettled
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: card3-real-claim-substrates-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: darn-seed-bound-card13-vs-card12
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card6-merge-base-error-stop-unstated
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised eleven findings; none recurs in round 2, so all are carried as resolved.
Round 2 raised four new findings: the gone-worktree teardown route never settling the claim (F1), the real claim substrates reached by no test (F2), card 13's darn-seed bound disagreeing with card 12's (F3), and the unstated merge-base error stop in card 6's whole-diff form (F4).
No BLOCKING finding remains; F1 is the only gating-class one and is first raised in round 2.
