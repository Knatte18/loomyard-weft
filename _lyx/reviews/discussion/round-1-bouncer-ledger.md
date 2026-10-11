---
round: 1
ledger:
  - key: d1-fixed-cache-ttl-premise
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d2-bound-overstates-probe-reach
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d11-proof-does-not-isolate-hop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: failed-ping-wrong-way-forward
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: ping-vs-reviewer-liveness-hold
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d5-slug-from-other-task-worktree
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d10-in-flight-claim-wrong-runs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: uncapped-ping-count-off-by-one
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1, first ledger.
The blocking issue is D1's premise: the prompt-cache lifetime is a per-role shuttle setting, so the ping interval and its pricing rationale need a role-TTL alternative and a provider-seam-clean source.
D2's Bound and D11's proof claim more than the code and files show.
The remaining findings are wording and coverage gaps in D1, D5, D10 and the ping count.
