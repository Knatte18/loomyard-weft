---
round: 4
ledger:
  - key: card20-coverage-citation-unsupported
    rounds: [1, 2, 3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: card17-orch-stencil-check-outcome-unstated
    rounds: [1, 2, 3, 4]
    status: open
    class: scope
    severity: MEDIUM
  - key: card21-watcher-table-underspecified
    rounds: [1, 2, 3]
    status: resolved
  - key: card20-skillunknown-live-path-dropped
    rounds: [2]
    status: resolved
  - key: card21-startup-fold-row-shape-missing
    rounds: [2, 3]
    status: resolved
  - key: card20-fakes-target-and-impact-inaccurate
    rounds: [2]
    status: resolved
  - key: card17-changed-facts-unpinned
    rounds: [2, 3]
    status: resolved
  - key: card17-friction-dir-uses-missing
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
---

Round 4 raised three findings, none design-class.
F1 (MEDIUM, scope) is the run-loop bullet in card 17: `lyx batten pause` and `lyx batten goto` leave the run paused with no resume step, the same shape as the earlier unchecked-resume finding, so it continues `card17-orch-stencil-check-outcome-unstated`.
F2 (LOW, consistency) is the new key `card17-friction-dir-uses-missing`: `internal/loomengine#LoomFrictionDir` is missing from card 17's `Uses`.
F3 (LOW, consistency) is card 20's reason for keeping the `ContextTokens` delegation test, which does not separate it from the deleted `CompactedSince` test; it continues `card20-coverage-citation-unsupported`.
Card 21's watcher table and startup fold row shape, and card 17's unpinned changed facts, no longer appear and are resolved.
