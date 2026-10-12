---
round: 2
ledger:
  - key: vacuous-unknown-field-undecodable-fixtures
    rounds: [1]
    status: resolved
  - key: stale-strict-decode-comment-batten-overview
    rounds: [1]
    status: resolved
  - key: missing-space-unusualfiles-doc-comment
    rounds: [1]
    status: resolved
  - key: smoke-failure-before-persist-stale-decode-assertion
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: smoke-died-driver-reads-log-before-driver-writes
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
---

Round 1 raised three non-gating hygiene findings after the tolerant-decode change; round 2 did not re-raise them, so they are carried as resolved.
Round 2 raised two BLOCKING design findings from the Publish failure, both in `internal/loomcli/smoke_test.go`.
F1 maps to the stale "decode" assertion key: the poisoned-state rig now yields an invalid-state refusal.
F2 maps to the one-shot driver-log read key: the start handshake returns halted on its first poll, before the driver logs.
