---
round: 4
focus:
  - "decision-record.md D6 and its acceptance criteria: when a package's test files split across the `integration`, `tmux` and `llm` tags, does the record say where helpers shared across tags live, and does it require each tag to compile on its own? Settle whether a combined `go vet -tags integration,tmux,llm ./...` could pass while a single-tag build fails."
  - "decision-record.md D6 Bound and D10: does every sentence claiming a gate runs the `tmux` tier match what the record decides about which verify commands this task can set and which are left to the stencil text and `test-tiers`?"
  - "decision-record.md Constraints, the test-isolation line: does it say what happens to the `testmain_test.go` files tagged `!integration` that pair with a `testmain_integration_test.go`, once `tmux` and `llm` tags exist, and does the check cover every tag combination?"
---

Round 4 checks the text added in round 3 against the three open concerns.
