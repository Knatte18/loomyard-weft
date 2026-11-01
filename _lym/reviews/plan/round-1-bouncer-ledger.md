---
round: 1
ledger:
  - key: run-shell-c-not-format-expanded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card5-misses-smoke-selvage-status-assertion
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card4-parent-rows-stay-on-prime-branch
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: strand-flag-covers-every-agent-window-entry
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: gone-directory-check-after-prompt
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: real-tmux-test-repeats-tier0-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: strand-flag-bound-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
The one BLOCKING finding concerns the click binding and the confirmed line: tmux `run-shell` expands its command but not `-c`, so both start outside the run directory.
Its fix changes the approach (a change-directory statement in the pinned line) and asks the parent to amend the decision record, hence `design-wrong`.
The remaining six are local corrections whose result the finding text fixes.
