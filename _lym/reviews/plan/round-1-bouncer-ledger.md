---
round: 1
ledger:
  - key: orch-entry-overview-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: one-overview-comment-sweep-gap
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: relaunch-row-names-ignored-cmd
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, none gating.
F1 (`orch-entry-overview-doc-stale`): card 3 leaves the orch entry of `docs/overview.md` stating the replaced D4 behaviour.
F2 (`one-overview-comment-sweep-gap`): the comment sweep misses four singular-overview statements in reed and orchcli.
F3 (`relaunch-row-names-ignored-cmd`): card 1's relaunch row names a command the IfAbsent relaunch ignores.
Each fix is a local text correction the finding already spells out.
