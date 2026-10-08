---
round: 3
ledger:
  - key: d7-followup-card-needs-overview-index
    rounds: [1, 2]
    status: resolved
  - key: d1-report-head-lock-bound-by-trust
    rounds: [1]
    status: resolved
  - key: d6-card-amended-mark-lost-on-failed-recovery
    rounds: [1, 2]
    status: resolved
  - key: d3-archive-only-guards-unstated
    rounds: [1]
    status: resolved
  - key: d3-d4-reset-start-tree-disagree
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: d10-dead-remedy-contradicts-master-ladder
    rounds: [1]
    status: resolved
  - key: d10-config-invalid-unbounded
    rounds: [1]
    status: resolved
  - key: d12-migration-both-keys-and-key-levels
    rounds: [1]
    status: resolved
  - key: scope-d8-acceptance-disagree
    rounds: [1]
    status: resolved
  - key: d13-numeric-inputs-no-usable-source
    rounds: [1]
    status: resolved
  - key: auto-mode-narrative-and-open-risk-misplaced
    rounds: [1]
    status: resolved
  - key: d7-begun-index-line-no-mechanism
    rounds: [2]
    status: resolved
  - key: d3-start-target-archive-trigger-set-open
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d7-acceptance-pins-fewer-regions-than-decision
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: d1-guard-claim-conflicts-with-d3-archive-only
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 3 raised four findings, none `design` and none BLOCKING.

F1 (MEDIUM, `decision`) is new: D3 does not close which states of the start target archive without moving, namely a recorded start missing from the repository and recorded starts with no common ancestor.
F2 (LOW, `consistency`) continues `d3-d4-reset-start-tree-disagree`: D4 and the Acceptance line disagree on which resets list the paths they left.
F3 (LOW, `consistency`) is new: the D7 Acceptance line names fewer pinned regions than D7 does.
F4 (LOW, `consistency`) is new: D1's "every target keeps the guards" conflicts with D3's archive-only path.

The D7 follow-up-card key (BLOCKING in round 2), the begun-card index line key and the D6 `card_amended` key produced no finding in round 3 and are marked resolved; the review lists the D7 and D6 points as settled.
The earlier-round keys already resolved stay resolved.
