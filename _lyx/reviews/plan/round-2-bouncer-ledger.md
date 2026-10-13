---
round: 2
ledger:
  - key: fmt-file-args-parsed-as-run-id
    rounds: [1]
    status: resolved
  - key: fmt-nonexistent-path-refusal-unreached
    rounds: [1]
    status: resolved
  - key: leftover-executor-missing-from-guard-table
    rounds: [1]
    status: resolved
  - key: fmt-unreadable-file-refusal-unreached
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised three scope findings on cards 6 and 7; round 2 shows all three fixed.
The nonexistent-path half of the card 6 refusal now has a row.
Round 2 raised one scope finding (F1): the `planGoFormat` unreadable-file refusal has no `TestFmtCmd` row, and the reason for other `EvalSymlinks` failures is unstated.
The fix is a local test row plus one sentence, whose result the finding text states.
