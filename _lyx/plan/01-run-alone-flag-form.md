# Card 1 — run-alone-flag-form

**Edit:**
- `cmd/testtiming/redundancy.go`
- `cmd/testtiming/redundancy_test.go`

**Intent:**
D7, the tool's half.
`runAlone` starts the coverage binary with `-test.run <pattern>`, `-test.coverprofile <file>` and `-test.v` as separate arguments.
A test that runs a cobra command without `SetArgs` reads the process's own arguments, and the separate `<pattern>` and `<file>` values then show up as positional arguments, so the test fails alone while it passes under `go test`, which passes every flag in `-flag=value` form.
That is why nine `internal/loomcli` tests (`TestValidatePlanCmd`, the `TestGateParity_*` family, `TestRunCLI_GroupGuard_NoGitRepoNeeded` and others) are listed as "failed" in the committed report; running them by hand with `-test.run=^Name$ -test.v=true -test.coverprofile=<file>` passes all of them.

Build the argument vector in one small pure function in `redundancy.go` that emits every flag in the single-argument `-flag=value` form (`-test.run=^<quoted name>$`, `-test.coverprofile=<file>`, `-test.v=true`), and have `runAlone` use it.
Replace the comment in `runAlone` that says a test reading the process's arguments fails under the binary's flags with one that says why the `=` form is used.
Cover the function with a table row or test in `redundancy_test.go` asserting the exact argument vector, including a test name that needs `regexp.QuoteMeta`.
No change to `runAlone`'s result handling.

**ImpactSummary:** Changes only how the redundancy mode passes flags to each per-test coverage run; the timing mode and the report format are untouched.
