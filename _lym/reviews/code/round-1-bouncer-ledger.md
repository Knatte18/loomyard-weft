---
round: 1
ledger:
  - key: board-sync-recorder-replaces-real-spawnsync
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: ensureorwarn-quiet-success-unasserted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: scripted-notifier-installed-binary-wide
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: summary-net-test-line-figure-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: commit-body-omits-serial-failingseam-row
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so every key is new.

The open work centres on the test cut in `internal/hubreconcile`:
seams that `TestMain` now installs for the whole test binary (board sync, notifier),
and an `EnsureOrWarn` assertion removed with the retry walk it sat on.
The notifier question is a ruling for the run's parent, not a code change for the fixer.
The other two entries are accuracy points in the change description and a commit body.
