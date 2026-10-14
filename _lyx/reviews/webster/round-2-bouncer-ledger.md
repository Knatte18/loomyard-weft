---
round: 2
ledger:
  - key: status-help-omits-sessions-and-stale-processes
    rounds: [1]
    status: resolved
  - key: overview-caller-rule-omits-revive
    rounds: [1]
    status: resolved
  - key: command-runcli-doc-comments-predate-tender
    rounds: [1]
    status: resolved
  - key: shortcuts-bullet-missing-comma
    rounds: [1]
    status: resolved
  - key: handover-execs-unresolved-tmux-name
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised four documentation and help-text consistency findings, none gating; round 2 does not raise them again.
Round 2 raised one BLOCKING design finding: `handOver` execs the unresolved `tmux` name, which fails every real attach and causes the five failing reedcli smoke tests.
The finding text states the fix exactly (`exec.LookPath`, then the resolved path as both exec path and argv[0]), so it is a local correction.
