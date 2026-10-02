---
round: 2
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
    rounds: [1, 2]
    status: resolved
  - key: failed-send-cap-unspecified
    rounds: [2]
    status: resolved
  - key: finalize-runs-state-changing-rows
    rounds: [2]
    status: resolved
  - key: fresh-start-leaves-prior-round-open
    rounds: [2]
    status: resolved
  - key: notify-delivered-on-closed-request
    rounds: [2]
    status: resolved
---

Round 1 reviewed the pair after the hub review replaced D5's reed typing with a SendMessage sent by the writer.
Its seven findings, the follow-on of that swap, were all fixed.
The judge pass left one gap open: D5's throttle and cap and D6's `notify` depended on per-round state that D3 never placed on disk.

Round 2 confirmed that gap (F1, BLOCKING) and added four related findings.
F2: a failed send's effect on the cap was unspecified.
F3: `finalize` could run the closure's state-changing rows.
F4: a fresh start left the prior round open.
F5: `notify` and `delivered` had no refusal for a closed request.
The fixer closed all five.
The delivery record now holds the prompt count, the last-prompt time and the waiting-notify count.
A prompt counts when the closure hands it to the loop.
`finalize` opens no request and consumes no verdict.
A new round supersedes an earlier one that has no verdict.
`notify` and `delivered` refuse on an expired request, and `notify` also refuses on one with a verdict.

Nothing is open after round 2.
Two edge cases were considered and not raised.
One is a stale fork's verdict landing on a newer round, which D6's bound on unchecked verbs already covers.
The other is two processes writing one delivery record, which is a write mechanism for the plan.
