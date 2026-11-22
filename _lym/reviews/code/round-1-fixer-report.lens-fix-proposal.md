```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F3, F4, F5, F6, F7; F2 was fixed differently).

Findings:

1. **LOW** — F3 test, `internal/gatecli/tierrun_test.go` (`TestRunTierMemoryGuard`, `registered`/`commandPID`).
   - The `registered` flag is only set when the guard's `sample` runs while the command is held. In the guarding row, the command waits for the guard's kill file, and the kill only fires after a `sample`. So `sample` is guaranteed to run while the command is registered, and the pin holds.
   - The "removed after" half is weaker. `commands.holds(commandPID)` is checked after `runTier` returns, and `commandPID` holds the last pid `sample` saw. That is the tier command's pid, so the check is valid.
   - The one gap is that `guard.foreign = j.tierCommands.holds` in `reports.go`, the wiring F3 names, is still not pinned. The proposal covers only `add` and `remove`. The review's summary lists `guard.foreign` too, but its `fix` key does not.
   - Evidence: the reviewer's F3 summary versus its `fix` text. The fixer followed the `fix` text faithfully.

2. **LOW** — F7 test, `internal/gatecli/test_integration_test.go` (new subtest "a halted call whose key lists reports writes no report").
   - It asserts exit code 1 and that no report is found. Only part of the "returns only after the chain ended" half is pinned. `g.wait` returns on process exit, so a report written late by an unstopped chain would be a process that wrote after exit, and the test cannot see it.
   - Evidence: the assertion runs only after `g.wait`. The reviewer's proposal inherits this limit, and the no-write half is pinned.

3. **LOW** — F6, `internal/gatecli/reports_test.go`. With the stopped-chain row gone, nothing in the test code calls `chain.stop()`, and the `stop` path of `reportChain` has no direct test. The halt-path scenario in F7 covers it only indirectly. The reviewer proposed this deletion and accepted that cost. No breakage follows.

No BLOCKING or MEDIUM findings. The F1 and F5 doc rewrites agree with each other and with the `GateKey.Reports` doc. The F4 table matches the `lastTierStart` code: an empty plan gives -1, `[0,1,2,2]` gives 2, and `[2,0]` gives 0.

Tool calls made: none. My context already held the changed files and the review.
