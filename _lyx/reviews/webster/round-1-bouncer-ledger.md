---
round: 1
ledger:
  - key: boot-signal-test-not-discriminating
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: orch-hold-memory-not-forgotten-on-pass
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: orch-hold-stuck-survives-restart
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: idle-cpu-report-unmeasured-stat-tick-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: shuttle-wait-leftover-default-constant
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 (first round, no prior ledger).
F1 maps to `boot-signal-test-not-discriminating`: the reedcli boot-signal step passes via the backed-off cadence alone, so the signal-event branch is uncovered.
F2 and F3 map to the two orch watcher hold-memory keys.
F4 maps to the unsupported stat-tick figure in the idle CPU report.
F5 maps to the unread `defaultPollIntervalMS` and the stale 500ms wording in shuttle's wait file.
