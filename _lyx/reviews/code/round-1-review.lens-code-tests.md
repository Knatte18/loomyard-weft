```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Per behaviour**
- Later tier runs after a failing tier, exit is the first failure's code, closing line names every failed run: pinned by the scenario "a failing tier stops no later tier and the call exits with the first failure's code" (`test_integration_test.go` ~945). Its tier6 run-record check proves tier6 ran after tier0 failed.
- Guard step runs after a failing listed tier, with a failing guard run named in the closing line and the listed tier's exit code: pinned by the `guardTests` row "a failing tier still runs the guards…" (~1368), using the new `helperGuardExitEnv`.
- No closing line when every run passes: pinned by an added assertion in the existing two-tier passing scenario (~935).
- Refusal after a failed tier ends the call with the closing line and the refusal's code, "later runs did not run" suffix: pinned by the new scenario (~965).
- A timed-out tier no longer stops the guard step: pinned by the updated `scopes_integration_test.go` assertion (~226).
- `FailedTier` names the first failing tier: pinned by the new row "first of two failing lines" in `runlog_test.go` (~135).
- `haltExitCode`, case `end.ran && firstFailureCode == 0` (a cancelled context after a run that ended without failure exits 1 and prints "the call was cancelled while its last run ended without a failure"): none. Nothing in the gatecli tests references that message or `haltExitCode`.
- `haltExitCode`, case "busy hub after a failed run exits with that failure's code, not `SlotBusyExit`": none. The only `SlotBusyExit` tests (`test_integration_test.go:636`, `:776`) fail no earlier run.
- `haltExitCode`, case "a run that ran and halted after an earlier failure keeps the first failure's code": none.
- Test economy: the added scenarios are distinct behaviours and the rest are rows or assertions in existing tests, so I found no duplicate.

**Findings**
1. MEDIUM, design. `internal/gatecli/run.go:163-174` and `:151-157` add halt-path behaviour with no test: a halt is never green and exits 1, a busy hub after a failure exits with that failure's code, and a halt keeps the first failure's code. The two commits that fixed these halt paths (`2f19920f9`, `111243f83`) added no scenario for them. The doc's exit-code contract (`doc.go:253`) states these cases. Fix: add a scenario or table rows in `test_integration_test.go`. One would hold a slot busy after tier0 fails and assert the failure's code, not 75. One would cancel the call while the last run ends cleanly and assert exit 1 and the message.
2. LOW, consistency. `test_integration_test.go:155-160`: the stand-in picks `helperGuardExitEnv` by the `^(TestGuard` argument prefix, which couples it to the guard test names. This is acceptable and nothing needs fixing, so I note it only.

**Tool calls**
1. `git diff a60ec1b..HEAD` on the three test files: my context did not hold the test changes.
2. `grep` of the gatecli tests for the cancelled-call message, `haltExitCode` and `SlotBusyExit`: checking for coverage of the halt paths.
