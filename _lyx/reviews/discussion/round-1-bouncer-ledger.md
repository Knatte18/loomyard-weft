---
round: 1
ledger:
  - key: mid-turn-gate-sends-lose-reject
    rounds: [1]
    status: resolved
  - key: delivery-resend-count-unbounded
    rounds: [1]
    status: resolved
  - key: review-verbs-unchecked-no-bound
    rounds: [1]
    status: resolved
  - key: writer-stencil-write-boundary
    rounds: [1]
    status: resolved
  - key: reed-design-residue
    rounds: [1]
    status: resolved
  - key: delivery-address-two-names
    rounds: [1]
    status: resolved
  - key: process-history-in-decision-record
    rounds: [1]
    status: resolved
  - key: delivery-bookkeeping-not-persisted
    rounds: [1]
    status: open
---

Round 1 reviewed the pair right after the hub review had replaced D5 in place, swapping reed typing for the writer's SendMessage.
The seven reviewer findings were the follow-on of that swap.
They covered mid-turn sends that could lose a reject, an unbounded re-send loop, unchecked verbs with no stated bound, the writer stencil's write boundary, residue of the reed design, two names for one address, and process history in the decision record.
The fixer closed all seven, and the pair now settles turn-boundary sends, the three-prompt cap, D6's bound and D8's stencil carve-outs.

One gap remains open from the judge pass.
The per-round delivery state behind D5's throttle and cap and D6's `notify` has no home in D3's directory list.
So the stated bound on parent notices does not hold across a driver restart, and `notify` has no stated channel to the gate.
