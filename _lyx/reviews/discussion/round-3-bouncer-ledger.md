---
round: 3
ledger:
  - key: d7-failed-remote-update-and-diverged-state
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-lease-vs-ancestry-check-tip
    rounds: [1]
    status: resolved
  - key: d6-stuck-terminal-status-omitted
    rounds: [1]
    status: resolved
  - key: d3-relaxation-let-through-unstated
    rounds: [1, 2]
    status: resolved
  - key: dispatch-check-card-scope-inconsistent
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: LOW
  - key: d6-recover-batch-refuses-uncheckable-failed-batch
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-q11-dependence-no-fallback
    rounds: [1]
    status: resolved
  - key: d7-remote-update-order-before-local-checks
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d6-dead-batch-no-fork-premise
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-acceptance-omits-dead
    rounds: [2]
    status: resolved
  - key: d7-remote-tip-ancestor-of-reset
    rounds: [2]
    status: resolved
  - key: d1-deriveedges-mirror-cross-batch
    rounds: [2]
    status: resolved
  - key: d10-removed-assertion-unbounded
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: open-risks-repeats-d2-bound
    rounds: [2]
    status: resolved
  - key: support-log-contradicts-d7-on-telling-commits-apart
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: acceptance-criteria-miss-decided-behaviors
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: d1-card-order-assumes-contiguous-batches
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 1 raised seven findings, one BLOCKING; round 2 raised eleven, none BLOCKING; round 3 raises nine, one BLOCKING.
The D7 subject has been open since round 1 and escalated: the plain-merge way forward discards the commits it merged (F1), and the order of the remote update against local gate refusals, plus the offline and no-remote cases, is still undecided (F2).
The D6 subject has also persisted: `recover-batch` is named as the way forward for a failed batch with uncheckable paths (F3), and the restamp of a dead batch states no bound on a live old-card strand (F4).
D10 is now raised at MEDIUM because it describes a check the code does not perform (F5).
D3, the D6 acceptance omission of `dead`, D7's ancestor tip, D1's `deriveEdges` mirror and the duplicated Open risks bullet are resolved.
New at LOW: the support log contradicting D7 (F6), acceptance criteria missing decided behaviors (F7), D1's card-order contiguity assumption (F8), and D1's attributed card, carried on the existing `dispatch-check-card-scope-inconsistent` key (F9).
