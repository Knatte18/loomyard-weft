---
round: 3
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
    status: resolved
---

Round 1 raised four non-gating consistency findings, all resolved.
Round 2 raised one BLOCKING design finding, `handOver` execing the unresolved `tmux` name.
Round 3 found the fix in place and raised nothing, so every key is resolved.
