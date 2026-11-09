```yaml
lens: code-tests
outcome: reported
```

**Behaviour → pinning test**

- `--stop-on-fail` halts at the first failed tier: `TestGateTest_Scenario`, the new subtest. It covers the tier0 failure, the skipped tier6, the stop line and the exit code.
  - Build-blocked stop (`end.blocked` → label `build`): none.
  - A tier crashed twice does not stop the call (`stopsCall`'s crash exclusion): none.
  - The last run failing prints no stop line (`i+1 < len(plan)`): none.
  - `--upto` keeps its default of running every tier: covered only by the pre-existing tests of the unchanged path.
- `StopOnFailCall` appends the flag: no direct test.
  - `darnwiring.go` adds it only for `darnWriteGateKey`, to both the told command and the tree-scoped command.
  - No test checks that the Darn-Write key gets the flag and the other keys do not.
  - `wiring_darn_test.go` only gained a seam marker.
- `FailedTests` parsing: `TestFailedTests`, with rows for tier and guard, subtest folding, no report, outside any test, build, vet, timeout, mixed and cut reports.
  - Not pinned: a `lint` step header (`reportRunTier` rejects it, as with vet).
  - Not pinned: CRLF input.
  - Not pinned: duplicate de-duplication across two runs.
- `FailedTestsCall`: `TestFailedTestsCall` covers tier order, de-duplication, linux and windows quoting, an empty input and an unquotable package.
- The rerun flow inside the gate: `TestVerifyGate_DarnFormRunsTheFailedTestsAloneFirst`. Its four steps cover:
  - no rerun on the first arrival;
  - tests that fail again, with the note, and the full command not run;
  - tests that pass alone, then the full command;
  - a failure naming no test, which keeps nothing.
  - Dirty tree during the rerun (`StatusDirty` in `runFailedTestsAlone`): none.
  - Timed-out run or runner crash keeps nothing (`namedFailures`): none.
  - A rerun call that cannot be rendered falls back to the full command: none.
  - The round form (non-darn) ignores failed tests: none.
  - A rerun that fails naming no test: none. The standalone `alone-report` stub always names `TestA`.
  - The still-failed tests are kept for the arrival after (`failedBefore = stillFailed`): the test never gates a third arrival after a second alone failure.
  - The `Terminal: res.RunnerCrash` result: covered by the pre-existing `TestVerifyGate_TerminalOnlyForARunnerCrash`.
- Budget clock: `TestVerifyBudget` covers:
  - the notice sent once at an arrival past the limit, with its content;
  - the timer path, including the delay and the cancel;
  - nothing sent within the limit;
  - a zero limit or nil Notify turning it off.
  - A `Notify` error being logged and not retried: none.
  - The notice cut to `verifyNoticeBytes`, including the UTF-8 boundary: none.
  - `failed()` fallback to `exit code N` when the log is unreadable or has no closing line: none.
  - The clock being wired into `NewVerifyGate` (`arrive`, `arm`, `failed` calls): none. Every gate test passes `VerifyBudget{}`, so a deleted `clock.arrive` call would break nothing.
- Budget config `verify_budget_min`: `config_test.go`'s `templateConfig` now holds 30 and `configreg_test.go` lists the key.
  - `darnwiring.go` converting minutes to a duration and sending to `orchcli.NotifyParent`: none.
- Halt message and `EndedEarly`: `gateattempts_test.go` has two new rows.
  - Row 1: `Attempts == 0` with a cause.
  - Row 2: `Attempts > 0` with a cause.
  - `TestGate_RepromptOverPicker` has three rows that assert `EndedEarly` holds "no input box".
  - The re-prompt send failed cause (`handleGatedBoundary`): none.
  - The entry wait ran out cause (`wait.go`): none.
  - The default "session ended or deadline" cause, and `gateEndedEarly` returning "" on a pass, a terminal failure or a spent budget: none.
  - The second `shedadapters` row also asserts the file content, in a table whose neighbouring rows already do.

**Findings**

1. MEDIUM, design. The `shuttleengine` causes have no test. Location: `internal/shuttleengine/wait.go:408`, `internal/shuttleengine/run.go` (`gateEndedEarly`), `internal/shuttleengine/wait.go:630`. Only the no-input-box cause is asserted, in `gate_test.go:1319-1327`. Not covered:
   - the wait-expiry cause;
   - the send-failure cause;
   - the default session/deadline cause;
   - the empty result on a pass, a terminal failure and a spent budget.
   - Together these are the main logic of the issue #92 fix, and an unconditional `EndedEarly` would pass every current test.
2. MEDIUM, design. Nothing pins the budget wiring into the gate. Location: `internal/loomshed/verifygate.go` (`clock.arrive`, `clock.arm`, `clock.failed`), `internal/loomcli/darnwiring.go:459`. `TestVerifyBudget` drives `budgetClock` directly and every `NewVerifyGate` test passes `VerifyBudget{}`. Removing the budget calls from the gate, or the `verify_budget_min` to `Limit` conversion, fails no test. Add a gate-level row that tells a budget and checks that `Notify` fires.
3. MEDIUM, design. Nothing pins that the Darn-Write key alone gets `--stop-on-fail`. Location: `internal/loomcli/darnwiring.go:452-471`. This is the Goal's "verify gate defaults to stopping at the first failing tier". Dropping the wrapper, or applying it to the landing key, passes all tests. Add a `wiring_darn_test.go` assertion on `VerifyCommand("darn-write")`, `VerifyCommand` for another key, and `VerifyTreeScope`.
4. LOW, design. The rerun's failure branches are untested. Location: `internal/loomshed/verifygate.go` (`runFailedTestsAlone`, `namedFailures`). Untested: the dirty-tree outcome, `TimedOut`/`RunnerCrash` keeping nothing, a failed rerun that names no test falling through to the full command, and the render-error fallback. The `failedBefore` handoff on a second failure is also unpinned.
5. LOW, design. The `--stop-on-fail` exclusions in `internal/gatecli/run.go:191-196` are unpinned. A build-blocked stop and a runner crash that must not stop the call both have no row. Also unpinned: the last-run no-op.
6. LOW, design. Two `verifybudget.go` behaviours have no row: the notice cut at 1000 bytes on a rune boundary, and the `failed()` fallback to `exit code N` when the log is unreadable or lacks the closing line.

Test economy: the new tests are not redundant with existing ones. The two `shedadapters` rows, `TestFailedTests` and `TestFailedTestsCall` each pin distinct behaviour.

**Tool calls**
1. `git diff 7f21142e..HEAD -- '*_test.go'`, to read the test diffs, which my context lacked.
2. Read of the saved output of that diff, because the first output was truncated.
