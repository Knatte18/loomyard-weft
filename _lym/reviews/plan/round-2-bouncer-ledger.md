---
round: 2
ledger:
  - key: run-shell-c-not-format-expanded
    rounds: [1]
    status: resolved
  - key: card5-misses-smoke-selvage-status-assertion
    rounds: [1]
    status: resolved
  - key: card4-parent-rows-stay-on-prime-branch
    rounds: [1]
    status: resolved
  - key: strand-flag-covers-every-agent-window-entry
    rounds: [1]
    status: resolved
  - key: gone-directory-check-after-prompt
    rounds: [1]
    status: resolved
  - key: real-tmux-test-repeats-tier0-rows
    rounds: [1]
    status: resolved
  - key: strand-flag-bound-unstated
    rounds: [1]
    status: resolved
  - key: reed-button-groups-need-open-maps
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: extends-on-rewritten-lines-cards-2-and-9
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: bar-integration-test-header-says-two-line
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's seven findings are not raised again in round 2; the review's Rejected section re-probed the click-line directory, the parent rows, the status-option readers and strand handling and found them sound, so all seven are resolved.
Round 2 raises one BLOCKING design finding: the reed template ships `task` and `beam` as lists of mappings, and `configengine.Load` rejects an override that does not repeat the template entries, so card 2 needs open maps for the three button groups.
Its fix adds a `ConfigOpenMaps` function and touches `internal/configreg`, a new contract, hence `design-wrong`.
The two non-gating findings are an `Extends` versus `Edit` mislabel in cards 2 and 9 and a stale header comment in `bar_integration_test.go`.
