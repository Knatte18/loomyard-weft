---
round: 1
ledger:
  - key: pairless-run-reconcile-errors
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: remove-snapshot-not-taken-in-origin-mode
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: origin-task-branch-survives-cancel
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: owned-process-end-outside-cancel-phase
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: reconcile-moves-live-pair
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ended-verdict-undecided
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: unreachable-origin-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: start-can-seed-during-cancel
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: interrupted-remove-cannot-rerun
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: windows-end-detects-nothing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: status-lock-across-network-gate-hibernate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cleanupslug-lock-conflict
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: two-run-locks-one-name
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-misplaces-board-push
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: wrong-tackler-marker-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so the ledger starts here with 15 open keys.
Four are BLOCKING: the pair-less run that cannot be cancelled, the missing origin-mode snapshot under `--remove`, the origin task branch that blocks a fresh rerun, and the owned-process end that sits outside the declared cancel phase.
Seven more design keys are MEDIUM, one `decision` key is MEDIUM, and three `consistency` keys are LOW.
Nothing is resolved yet.
