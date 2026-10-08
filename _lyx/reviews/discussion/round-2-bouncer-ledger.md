---
round: 2
ledger:
  - key: d7-border-row-subtraction-false-premise
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-booter-session-order-and-nested-up-recursion
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d12-concurrent-booter-order-race
    rounds: [1]
    status: resolved
  - key: d15-jsonc-insertion-not-byte-identical
    rounds: [1]
    status: resolved
  - key: config-colors-decision-bolted-on
    rounds: [1]
    status: resolved
  - key: config-color-values-unbounded-reader-unnamed
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
  - key: d13-claude-color-names-vs-provider-seam
    rounds: [1]
    status: resolved
  - key: pre-change-pane-criterion-outruns-d2-d14
    rounds: [1]
    status: resolved
  - key: support-log-missing-parent-change
    rounds: [1]
    status: resolved
  - key: d13-color-refusal-at-load-blast-radius
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-lock-placement-and-cross-worktree-up
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d15-atomic-write-readonly-contradiction
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d15-marker-edge-cases
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 2 keeps the D7 pin-arithmetic problem (F1, BLOCKING) and the D12 ordering bound (F3, MEDIUM) open from round 1.
The round-1 color-values finding recurs in reduced form as F7 (unknown segment, unnamed reader).
The round-1 D12 race, D15 byte-identity, color-decision placement, provider-seam, pre-change-pane and support-log findings no longer appear and are marked resolved.
New in round 2: D13's load-time refusal (F2, BLOCKING), D12 lock placement (F4), D15 atomic write against read-only (F5) and D15 marker edge cases (F6).
