---
round: 3
focus:
  - "decision-record.md D6, D10, the Bound and the Acceptance criteria: settle whether the llm tier (compile-checked by `go vet -tags llm`, never run by a gate) is stated consistently, so that no sentence promises a gate runs a file retagged into llm, and the loss of those tests from any `-tags smoke` run is stated as intended and bounded."
  - "decision-record.md D6 Bound: settle whether it claims verify commands this task cannot set, given that card gates come from `cardGateCommand` in `internal/websterengine/cardgate.go`, which emits `go test -tags integration` and is out of this task's scope."
  - "decision-record.md D2 and its acceptance criterion on tests that spawn the built `lyx` binary or another subprocess: settle whether the way a spawning test is recognised is decided, and whether an unclassifiable test errs toward 'no coverage' rather than 'redundant'."
  - "decision-record.md Constraints: settle whether every PATTERN ID named as a rule to follow still exists after D8's retirements."
---

The previous round's fixes are new text, so each entry asks the reviewer to judge it fresh.
