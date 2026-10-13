---
round: 1
ledger:
  - key: fmt-file-args-parsed-as-run-id
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: fmt-nonexistent-path-refusal-unreached
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: leftover-executor-missing-from-guard-table
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three scope findings.
Card 6 (`lyx loom fmt`): the loom pre-run reads the first file argument as a run-id, so paths with a separator are refused before `RunE`, and the nonexistent-path refusal has no stated reason or test.
Card 7 (worktree leftover retry): `removeWorktreeLeftover` is absent from the mutation-record guard's executor table.
Each fix is a local correction whose result the finding text states.
