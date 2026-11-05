---
round: 1
ledger:
  - key: interactive-sessions-keep-memory-listagents-plugin-skills
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: listagents-denied-in-settings-not-launch-line
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: board-button-scribe-preload-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: no-scope-for-opened-session-outside-hub
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: unreadable-plugins-file-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: claudeconfigdir-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: readinstalledplugins-edge-branches-untested
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings and no ledger preceded it.
Four are MEDIUM `decision` findings, each an unrecorded parent correction deferred to the parent.
Three are test gaps in `scope_test.go`, with fixes the review text already fixes.
No finding is `design`, so none gates.
