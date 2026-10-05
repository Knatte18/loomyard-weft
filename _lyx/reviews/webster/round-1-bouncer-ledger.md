---
round: 1
ledger:
  - key: menucmd-comment-clause-break
    rounds: [1]
    status: open
  - key: config-long-ragged-line
    rounds: [1]
    status: open
  - key: basedir-join-duplicated
    rounds: [1]
    status: open
---

Round 1 raised three NIT consistency findings in `internal/configcli/configcli.go`: F1 maps to `menucmd-comment-clause-break`, F2 to `config-long-ragged-line`, F3 to `basedir-join-duplicated`.
None is gating.
