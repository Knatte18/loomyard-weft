---
round: 1
ledger:
  - key: cap-two-counters-store-decides
    rounds: [1]
    status: resolved
  - key: pending-final-lift-unkeyed-unbounded
    rounds: [1]
    status: resolved
  - key: fresh-spawn-beginround-breaks-way-out
    rounds: [1]
    status: resolved
  - key: approve-supersede-bound-not-mechanical
    rounds: [1]
    status: resolved
  - key: acceptance-skips-refusals
    rounds: [1]
    status: resolved
  - key: liveness-read-failure-unbounded
    rounds: [1]
    status: resolved
  - key: no-go-sends-constraint-too-broad
    rounds: [1]
    status: resolved
  - key: nested-pair-timed-pass-only-in-risks
    rounds: [1]
    status: resolved
  - key: rationale-reargues-rejected-alternatives
    rounds: [1]
    status: resolved
  - key: cap-three-budget-comparison-off-by-one
    rounds: [1]
    status: resolved
---

Round 1 raised ten findings, all on the decision record, and all ten were fixed before judgment.

The three blocking findings were all about Decision 5's parent-review cap.
The store count, not the engine's in-memory `attempts` count, now decides the cap through a terminal `GateResult`.
The Pending/Final guard lift is keyed on a `GateEntry` field set only by the parent-review wiring, and `Final` fails on an unconsumed reject.
Fresh-spawn round retention and the gate's latest-round read make the approve way-out and the halt-again path actually hold.

The medium findings tightened bounds and tests.
They define the halted-at-cap test as three conditions, add acceptance criteria for the refusals, make a failed liveness read hold, and narrow the no-Go-sends constraint to the orch session.

The low and nit findings moved the nested-pair timed pass into Decision 4's Bound, cut re-argued alternatives from two rationales without losing any requirement, and dropped an off-by-one comparison.

The judge approved round 1 with no findings left open.
