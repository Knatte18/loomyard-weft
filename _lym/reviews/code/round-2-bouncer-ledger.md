---
round: 2
ledger:
  - key: board-sync-recorder-replaces-real-spawnsync
    rounds: [1]
    status: resolved
  - key: ensureorwarn-quiet-success-unasserted
    rounds: [1]
    status: resolved
  - key: scripted-notifier-installed-binary-wide
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: summary-net-test-line-figure-stale
    rounds: [1]
    status: resolved
  - key: commit-body-omits-serial-failingseam-row
    rounds: [1]
    status: resolved
---

The work centres on the test cut in `internal/hubreconcile`.
Round 2 confirmed the fixes for the board-sync recorder, the `EnsureOrWarn` assertion, the change description's line figure and the commit body.

The one open key is the notifier recorder that `TestMain` installs for the whole test binary.
It is a ruling for the run's parent, not a code change for the fixer, and the decision record holds no added entry that settles it.
