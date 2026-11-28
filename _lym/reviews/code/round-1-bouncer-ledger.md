---
round: 1
ledger:
    - key: parallel-group-runs-global-logger-capture
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: rows-doc-comment-ordering-rule-too-broad
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: measurements-setup-difference-unnamed
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
---
Round 1 opened three keys.

- `parallel-group-runs-global-logger-capture` (F1) is the gating one:
  a step using `logcapture.Capture` sits in a parallel group of `TestBattenIntegration_Rows`, against `PATTERN-test-speed`.
- `rows-doc-comment-ordering-rule-too-broad` (F2): the doc comment's rule on steps that leave the hub changed does not match the table.
- `measurements-setup-difference-unnamed` (F3): the change description's `## Measurements` names no command, tier or set-up difference.
