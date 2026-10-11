---
round: 3
ledger:
  - key: d1-fixed-cache-ttl-premise
    rounds: [1]
    status: resolved
  - key: d2-bound-overstates-probe-reach
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d11-proof-does-not-isolate-hop
    rounds: [1, 2]
    status: resolved
  - key: failed-ping-wrong-way-forward
    rounds: [1]
    status: resolved
  - key: ping-vs-reviewer-liveness-hold
    rounds: [1]
    status: resolved
  - key: d5-slug-from-other-task-worktree
    rounds: [1]
    status: resolved
  - key: d10-in-flight-claim-wrong-runs
    rounds: [1]
    status: resolved
  - key: uncapped-ping-count-off-by-one
    rounds: [1]
    status: resolved
  - key: scope-only-judging-decided-and-unconfirmed
    rounds: [2]
    status: resolved
  - key: ledger-interval-location-contradicts-d1
    rounds: [2]
    status: resolved
  - key: d1-interval-precedent-and-never-pinged
    rounds: [2]
    status: resolved
  - key: d2-rationale-finalize-sole-expiry-claim
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eight findings; D1's cache-lifetime premise was the blocker and its fix held.
Round 2 reopened D2's Bound and D11 through the fixes themselves.
Round 3 leaves two findings and no BLOCKING one.
D11 and the three round-2 consistency keys were not raised and are resolved.
D2's Bound is the one key that has been open in all three rounds: each fix added a new claim about probe reach or resume, and the latest one (a resume of the Stuck-halted run) has no route in the code, since finalize removes the strand on any `OutcomeDone` and the next start is a fresh spawn beginning round N+1.
The new LOW key records that D2's Rationale calls finalize the sole place expiry is marked, while the gate closure's expiry branch marks it too.
