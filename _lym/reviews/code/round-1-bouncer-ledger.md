---
round: 1
ledger:
  - key: drift-baseline-docs-rule-omitted
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: orch-stencil-names-claude-md
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parent-directive-background-one-rule-behind
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three non-gating findings.
The drift baseline stencil omits the docs rule the board entry asked for, on the record's own D2 claim; the parent decides it.
The orch stencil still names `CLAUDE.md` in its placement sentence.
The `PATTERN-parent-directive` background file still calls the edit-directive stencil the only statement of one rule.
