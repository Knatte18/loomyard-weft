---
round: 2
focus:
  - "Stencils `contracts/stencils/darn/darn-fragment-make-change.md` and `contracts/stencils/webster/webster-body-implementer.md`, and `noTestNotice` in `internal/gatecli/tierrun.go`: does any of them still tell an agent to run a raw `dotnet test` as an unconditional step, where a hub whose `refuse_raw` lists `dotnet test` has its shell guard refuse it (`PATTERN-refusal-way-forward`)?"
  - "`noTestNotice` and `runInvocations` in `internal/gatecli/tierrun.go`: does a multi-project invocation, such as a whole-run solution where one test project disables itself and another runs tests, print a line naming the skipped project or class, or state that the cause cannot be seen, with `internal/gatecli/doc.go` stating which skips the line covers?"
  - "`TestRunTierNoTestNotice` in `internal/gatecli/tierrun_test.go` and the `CodeLocation` branch in `internal/fabricengine/weftwiring.go` that returns a records location unchanged when its paired code directory is absent: is each branch of the no-test line (failing invocation, no targets) and of `CodeLocation` pinned by a test that no other test covers?"
---

Focus targets the round 1 gating findings and the coverage gaps beside them.
