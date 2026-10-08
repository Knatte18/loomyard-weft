---
round: 3
ledger:
  - key: d7-border-row-subtraction-false-premise
    rounds: [1, 2]
    status: resolved
  - key: d12-booter-session-order-and-nested-up-recursion
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
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
    status: resolved
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
    status: resolved
  - key: d12-lock-placement-and-cross-worktree-up
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d15-atomic-write-readonly-contradiction
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: d15-marker-edge-cases
    rounds: [2]
    status: resolved
  - key: d7-top-pin-row0-source-undefined
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-top-pin-ignores-refused-border-status
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-top-pin-one-row-floor
    rounds: [3]
    status: open
    class: design
    severity: LOW
  - key: d8-runtime-format-single-quote-bound
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 3 closes the D7 false-premise finding: the pin value is right for a collapsed strip.
D7 now has three new defects: which pane is at window row 0 (F1), the unconditional subtraction when `pane-border-status` was refused (F2), and no floor for a one-row top cell (F3).
The D12 ordering key reduces to a LOW consistency finding about how the stop is stated and tested (F5).
The D12 lock and teller-input finding recurs as a MEDIUM `decision` finding about what the teller supplies (F4).
The D15 atomic-write finding recurs at LOW (F7).
D8's single-quoted runtime format values are new (F6).
The color-values, D13 load-refusal and D15 marker findings no longer appear and are marked resolved.
