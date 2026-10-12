---
round: 1
ledger:
  - key: d10-handoff-keeps-newest-not-lasthandoff
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d9-scratch-deletes-lock-files-and-races-batten-run
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d9-tracked-but-deleted-no-done-check
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-mode-0700-fails-on-windows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-windows-redirect-and-removal-failure-undecided
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: goal-no-testmain-packages-leak-on-kill
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-allowlist-unbounded-and-disagrees-with-q3
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-in-process-segments-skip-verb-refusals
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d11-board-read-failure-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d11-scratch-sole-declarer-claim-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-sweep-check-order-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d3-adopted-root-no-trust-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d9-orphan-rule-fails-open-on-stat-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 raised thirteen findings against the discussion pair, all still open.
The two BLOCKING ones sit in the orch and batten cleanup segments: handoff retention must follow `State.LastHandoff`, and batten scratch removal must keep every lock file and hold the status lock.
The remaining MEDIUM design findings concern the done check on tracked-but-deleted runs, the Windows mode check, test packages without a `TestMain`, the scan allowlist bound and the board-read failure.
