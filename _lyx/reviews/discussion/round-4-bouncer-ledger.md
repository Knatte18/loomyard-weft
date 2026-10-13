---
round: 4
ledger:
  - key: selfexec-leaf-set-outside-pattern
    rounds: [1]
    status: resolved
  - key: replace-selection-moves-viewer-off-shell
    rounds: [1]
    status: resolved
  - key: hidden-window-insert-target-missing
    rounds: [1]
    status: resolved
  - key: no-shell-fallback-order-unbounded
    rounds: [1]
    status: resolved
  - key: migration-step-failure-bound-missing
    rounds: [1]
    status: resolved
  - key: stale-sidecar-consequence-only-in-support-log
    rounds: [1]
    status: resolved
  - key: scope-out-carries-history
    rounds: [1]
    status: resolved
  - key: migration-criterion-says-last
    rounds: [1]
    status: resolved
  - key: attach-selection-rides-suppressed-chain
    rounds: [2]
    status: resolved
  - key: migration-warn-recurs-each-open-and-attach
    rounds: [2, 3]
    status: resolved
  - key: newest-agent-premise-unqualified-on-psmux
    rounds: [2]
    status: resolved
  - key: strand-window-fallback-renames-overview-or-agent
    rounds: [3, 4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: attach-select-moves-other-attached-clients
    rounds: [3, 4]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: d1-lowest-index-unconditional-on-psmux
    rounds: [3]
    status: resolved
  - key: migration-criterion-omits-selvage-alone-and-no-size
    rounds: [3]
    status: resolved
  - key: log-levels-constraint-cites-d9
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 4 raises no design-class finding and no BLOCKING one.
The round 3 Warn-recurrence key, the D1 lowest-index key and the migration-criterion key are not raised again and are resolved.
The two round 3 design keys narrow: the strand-window fallback survives only as a D10 parenthetical that contradicts D3 (LOW consistency), and the attach selection survives only as the failed-client-listing case (MEDIUM decision).
A new NIT notes that the `PATTERN-log-levels` Constraint cites D9, which sets no level.
Each remaining fix is a local edit the review text supplies.
