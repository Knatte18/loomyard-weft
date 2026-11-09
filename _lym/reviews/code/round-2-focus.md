---
round: 2
focus:
  - "internal/shuttleengine/wait.go and gate_test.go: does a test assert GateOutcome.EndedEarly names each cause (entry-wait expiry, re-prompt send failure, plain session end, no input box) and stays empty on a pass, a terminal failure and a failure whose entry spent its attempts?"
  - "internal/loomcli/darnwiring.go and wiring_darn_test.go: does a test pin that only the darn-write key carries --stop-on-fail on both VerifyCommand and VerifyTreeScope, and that the landing key does not?"
  - "internal/loomshed/verifygate.go, verifybudget.go and darnwiring.go: does a test drive NewVerifyGate with a short VerifyBudget Limit, an injected clock and a recording Notify, and pin the verify_budget_min to Limit minutes conversion?"
  - "internal/shuttleengine/wait.go: does each of finalizeAndRemoveDiedWindow and gateEndedEarly have exactly its own doc comment directly above its func line?"
  - "internal/loomshed/verifygate.go: are the failed-tests rerun branches (dirty tree, timeout or crash, no named test, render error), the build-blocked and doubly crashed stop-on-fail cases, and the notice cut and exit-code fallback each covered by a test row?"
  - "internal/loomshed/verifybudget.go: does the budget notice carry the gate's failures to date, or does the decision record state that the latest failure suffices?"
---

Round 1 left open test-coverage gaps, a doc comment placement and the notice scope; round 2 checks the fixes and any new text they added.
