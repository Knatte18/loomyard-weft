---
round: 2
ledger:
  - key: d7-failed-remote-update-and-diverged-state
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-lease-vs-ancestry-check-tip
    rounds: [1]
    status: resolved
  - key: d6-stuck-terminal-status-omitted
    rounds: [1]
    status: resolved
  - key: d3-relaxation-let-through-unstated
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: dispatch-check-card-scope-inconsistent
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d6-recover-batch-refuses-uncheckable-failed-batch
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-q11-dependence-no-fallback
    rounds: [1]
    status: resolved
  - key: d7-remote-update-order-before-local-checks
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d6-dead-batch-no-fork-premise
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-acceptance-omits-dead
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d7-remote-tip-ancestor-of-reset
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: d1-deriveedges-mirror-cross-batch
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d10-removed-assertion-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: open-risks-repeats-d2-bound
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised seven findings, one BLOCKING, all against `decision-record.md`.
Round 2 confirms three of them resolved: the lease-versus-ancestry tip (D7), the missing `stuck` status (D6) and the D9 dependence note.
The D7 subject stays open at MEDIUM: the merge way forward does not hold for the state the bug reports, and the order of remote update and local checks leaves the failure outcome open.
D3's bound still misstates what bounds the residue, D4 and D6 still name a recover-batch way forward that fails for uncheckable paths, and the card-scope phrase still differs from the existing dispatch scoping.
New in round 2: D6's no-fork premise for a dead batch, the D6 acceptance criterion omitting `dead`, D7's remote tip behind the reset commit, D1's `deriveEdges` mirror across same-batch pairs, D10's unbounded assertion removal, and a duplicated Open risks bullet.
