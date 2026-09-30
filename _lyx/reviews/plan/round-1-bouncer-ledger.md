---
round: 1
ledger:
  - key: card2-createweftworktree-shared-with-reconcile
    rounds: [1]
    status: resolved
  - key: card2-no-checkout-index-stat-data
    rounds: [1]
    status: resolved
  - key: card3-deps-go-header-untargeted
    rounds: [1]
    status: resolved
---

Round 1 raised three findings, and the fixer resolved all three before this judgment.
The blocking one was card 2 changing `createWeftWorktree`, which Reconcile's dormant-weft path also calls.
The drop now lives in a new `forkrecords.go` function that only `Add` calls, and Reconcile is unchanged.
The two low findings were fixed too: card 2's write-out now keeps index stat data (`checkout-index -a -u`, `read-tree -m HEAD`), and card 3 no longer edits the untargeted `deps.go` header.
No finding is open.
