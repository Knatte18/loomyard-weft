---
round: 1
ledger:
  - key: done-run-clear-leaves-task-branch
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: docs-describe-old-done-run-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: tick-placement-and-landed-link-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: failed-counted-landings-update-leaves-stale-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-recipe-commit-message-unasserted
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, one per key above (F1 to F5 in order).
F1 is BLOCKING: the clear removes records and scratch but not the run's task branch, which the board entry asks for.
F2 is stale docs in `internal/beamcli/doc.go` and `docs/overview.md`.
F3 to F5 are LOW: unpinned tick placement, a stale `CountedLandings` entry after a failed update, and an unasserted commit message.
