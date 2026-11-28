---
round: 1
ledger:
    - key: stencil-test-pins-static-prose
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: measurement-rule-no-parent-path-drops-deliverable
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: ungated-subagents-bypass-measurement-rule
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: measurement-rule-scope-ambiguous-for-mixed-entry
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
---
Round 1 opened every key.

- `stencil-test-pins-static-prose` (F1): new assertions in `internal/darnengine/spec_test.go` pin a sentence both darn fragments carry, against `PATTERN-test-speed`.
- `measurement-rule-no-parent-path-drops-deliverable` (F2): with no parent or no answer, the writer skips the measurements and hands off as if done, and naming the gate calls in the change description contradicts Step 5.
- `ungated-subagents-bypass-measurement-rule` (F3): in the ungated fragment, subagents get the per-package gate rule without the parent-first rule.
- `measurement-rule-scope-ambiguous-for-mixed-entry` (F4): the rule does not say whether an entry's ordinary gate run still happens alongside its measurement calls.
