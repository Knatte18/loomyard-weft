---
round: 2
focus:
  - "internal/gatecli: confirm each gate refusal row in the `## gate` section of `contracts/specs/refusal-spec.md` (an unreadable or invalid `gate.yaml`, a `go` binary that cannot be started, a gate slot that cannot be acquired, a busy slot, no package, a missing -C directory) has a test that reaches it, and that no test calls an unexported helper such as `goTestArgs` directly where the CLI scenario in `test_integration_test.go` already asserts the same argument list."
  - "internal/websterengine and internal/landingshed: for every test added in the range, settle whether it calls `t.Parallel` or carries a comment naming the process-global state it touches, per `PATTERN-test-speed`."
  - "internal/loomshed `checkedPublishFailure` and `failedTestPackage`, internal/gateslot `LoadConfig`, internal/configcli `runMenu`: settle whether a record's log path file name and test package can reach the review prompt or the `go` command line unchecked, whether an absent `gate.yaml` is recognised without matching error text, and whether a test fails when the menu's hub-wide write guard is not set."
---

Round 1 left these areas open; the entries name where to look and which question to settle.
