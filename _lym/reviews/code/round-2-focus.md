---
round: 2
focus:
  - "In internal/landingshed (finalize.go judgeFixerChange, fixerattempts.go, fixaccept.go AcceptFixReview), settle whether an operator's acceptance of a fix review halt is bound to the task HEAD it accepted, so a fixer turn or operator commit made after the accept and before the resume still reaches the cap check and the fix review."
  - "In internal/landingshed/finalize_gate_test.go, settle whether the resume path with a pending fix review report is pinned: the gateAndFix branch that runs the fixer on a passing gate, and runFixer clearing the pending report after a fixed end."
---

Focus targets the code paths touched by any fix to the acceptance record and the resume tests.
