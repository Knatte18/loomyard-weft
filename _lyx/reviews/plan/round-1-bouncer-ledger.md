---
round: 1
ledger:
  - key: resolver-removed-parent-worktree-errors
    rounds: [1]
    status: resolved
  - key: final-closure-consumed-reject-reason
    rounds: [1]
    status: resolved
  - key: inmemory-vs-store-budget-claim-uncarried
    rounds: [1]
    status: resolved
  - key: liveness-error-warn-unbounded
    rounds: [1]
    status: resolved
  - key: verdict-recorded-way-forward-at-cap
    rounds: [1]
    status: resolved
---

Round 1 was the first round, so no earlier ledger existed.
The review raised five findings, and the fixer resolved every one in the plan text:

- Card 2's resolver errored when the parent worktree had been removed.
- Card 5's final-closure reason was wrong for a consumed reject.
- No card carried the claim that the in-memory count stays within the store count.
- Card 6's liveness-error Warn had no rate bound.
- Card 10 left the verdict-recorded way forward wrong at the cap.

Nothing is open.
