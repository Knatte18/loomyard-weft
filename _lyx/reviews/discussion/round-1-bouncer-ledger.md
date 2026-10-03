---
round: 1
ledger:
  - key: remote-landed-rule-counts-own-tracking-ref
    rounds: [1]
    status: resolved
  - key: sibling-outside-pathspec-refusal-after-shutdown
    rounds: [1]
    status: resolved
  - key: retiring-mark-bound-overstated
    rounds: [1]
    status: resolved
  - key: parked-driver-ended-unbounded
    rounds: [1]
    status: resolved
  - key: not-found-ignores-stray-path
    rounds: [1]
    status: resolved
  - key: warp-sha-pick-under-open-risks
    rounds: [1]
    status: resolved
  - key: goal-overstates-branch-removal
    rounds: [1]
    status: resolved
  - key: d10-narrative-belongs-in-support-log
    rounds: [1]
    status: resolved
---

Round 1 raised two blocking findings, one medium, four low and one nit.
The blocking findings were these:

- The remote landed rule let a branch's own `refs/remotes/origin/<branch>` satisfy the gate, so D8 and D9's deletions had no effective bound.
- D6's residual sibling-dirtiness refusal fired inside `Remove` after D4 had already ended the session, which broke D3's guarantee.

The medium finding was that D7's bound overstated what the retiring mark proves.
The low and nit findings were these:

- D2 said nothing about the parked case.
- D5's not-found error dropped a stray path.
- A decided warp-SHA pick sat under Open risks.
- The Goal promised removal of every branch.
- D10 carried narrative that belongs in the support log.

The round-1 fixer resolved all eight in the decision record and support log, and the judge confirmed each fix against the current text.
No finding remains open.
