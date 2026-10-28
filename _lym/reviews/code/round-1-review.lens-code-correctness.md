```yaml
lens: code-correctness
outcome: reported
```

**Correctness list (code-correctness lens): no findings.** Each case below was run by hand against the diff.

- `TestMain` dispatch (`internal/gitrepo/testmain_test.go`):
  - A child spawned with `-test.run=^$` and `GITREPO_TEST_PUSH_DIR=<clone>` reaches `TestMain`, runs `HermeticGitEnv`, finds the variable set, and exits with the body's code before `m.Run` and flag parsing. Correct.
  - A failed push returns 1 and writes to stderr, so `CombinedOutput` returns a non-nil error and the parent reports it. Correct.
  - The lock-holder child writes `LOCK-HELD` to unbuffered stdout and blocks. The `for {}` ends the function, so it compiles with no return. The parent's `StdoutPipe` scanner sees the marker, and `Kill` makes `Wait` return. Correct.
  - `init` in the tier1 file writes into a map declared in the untagged `testmain_test.go`. Package-level variables initialize before `init`, and under tier0 the map stays empty. Correct.
  - Inherited variable: if a variable is already set in an unrelated test process, that process runs the child body and exits 0 with no tests. Only a manually exported variable triggers this; it is not a defect.
  - Both variables set at once: map iteration order is random, so the body that runs is arbitrary. The parent spawns each child with only one variable set, so this does not occur.
- `TestCrossProduct` (`internal/fabricengine/livestate_matrix_test.go`):
  - `isOmitted` is a pure boolean. An omitted cell is counted and generates no subtest, as before.
  - An `Omissions` row naming a cell outside `effectiveStates` makes `omitted` differ from `len(Omissions) * len(anchors)`, so the tally still fails.
  - `ran` is still compared with the table-derived count. Correct.
- Short-circuit on `runtime.GOOS != "windows" && !t.Run(...)` (`internal/lymcwd/lymcwd_test.go`, `internal/reedcli/smoke_teardown_test.go`): on Windows the condition is false, so the function does not return early. On other OSes it behaves as `if !t.Run`. Correct.
- Table filters (`internal/reedengine/lifecycle_test.go`, `internal/reedcli/smoke_lifecycle_test.go`, `internal/reedcli/watchdog_test.go`, `internal/orchcli/rules_test.go`, `internal/websterengine/rebaseline_test.go`, `internal/websterengine/runlevel_test.go`):
  - The `||`/`&&` precedence in the `reedengine` filter is right.
  - The `setup` nil guard exists for the rows whose `setup` was removed.
  - The filters drop exactly the rows the old skips skipped on Windows. Non-Windows runs keep every row.
  - Root skips remain.
  - Callers of `makeUnreadable` and `skipAsRoot` are `rebaseline_test.go:363`, `rebaseline_test.go:377` and `runlevel_test.go:900`. All three sit in rows flagged `permissionBits`, so no unflagged caller would run `chmod` on Windows.
- `posixOnly` (`internal/pairteardown/archive_test.go`): a nil `run` is skipped by the loop before `t.Run`. On other OSes the wrapper returns the original function. Correct.
- `configcli` and `loomcli` `fmtverb` blocks: wrapped in `if runtime.GOOS != "windows"`, the root skip is kept, and the bodies are unchanged apart from indentation. Correct.
- Deleted `verifygate` subtest: it skipped before running anything, so its `t.Run` always returned true. Removing the `if !t.Run(...) { return }` wrapper does not change the control flow of the later subtests. Correct.
- `TestLoopLockOwner` moved to `internal/loomcli/landingdeps_linux_test.go`: the empty start-time case now fails with `Fatalf` instead of skipping. The move is correct on Linux.
- Build check: `go vet` is clean on Linux for tier0, tier1 and tier2 over every touched package. The Windows vet for `loomcli`, `websterengine` and several other touched packages could not run, because it needs cgo (tree-sitter and quarry).
- Windows vet of `internal/reedengine` under tier2 reports `mustTmuxOutput` undefined at `contract_integration_test.go:842`. That helper is in `switch_integration_test.go`, tagged `tier2 && linux`. Neither file is in this diff and both are unchanged from the base, so this predates the change and is not a finding.

Tool calls:
1. `grep makeUnreadable|skipAsRoot` in `internal/websterengine`: confirmed that every remaining caller sits in a Windows-filtered row, which the diff context did not show.
