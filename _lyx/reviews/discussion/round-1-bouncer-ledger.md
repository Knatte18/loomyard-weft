---
round: 1
ledger:
  - key: d7-border-row-subtraction-false-premise
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-booter-session-order-and-nested-up-recursion
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-concurrent-booter-order-race
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d15-jsonc-insertion-not-byte-identical
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: config-colors-decision-bolted-on
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: config-color-values-unbounded-reader-unnamed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d13-claude-color-names-vs-provider-seam
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: pre-change-pane-criterion-outruns-d2-d14
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: support-log-missing-parent-change
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first round, so no prior ledger exists.
Nine findings open: two BLOCKING design findings on the border-row budget (D7) and the boot order of sessions (D12), plus a MEDIUM design finding on D12's concurrency claim and one on unvalidated config colors.
The remaining findings are decision and consistency gaps around the JSONC insertion bound (D15), the config-colors decision's place in the record, the color-name location versus the provider seam, the pre-change-pane criterion, and the support log.
