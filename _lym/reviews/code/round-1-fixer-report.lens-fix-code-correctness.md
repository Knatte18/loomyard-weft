```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code through the correctness lens (walk each changed branch with a concrete input; check callers and that each added test fails with the defect in place).

No findings.

Walkthroughs:
- **Webster gate:** a first-run result of `Failed, ExitCode 134, RunnerCrash` is handled before `classifyHang`, so the exit-code-128-or-above hang rule cannot claim it. `failRunnerCrash` returns `Terminal`, so no re-prompt and no count increment. No pre-fix head or report is written. `verifyGateStuckReason` takes the crash prefix branch and ends in reentry alone. The `gate.Reason` consumers are only that function and `shedadapters/gateattempts.go:30`, which passes the reason through, so no other caller breaks.
- **`retryTier`:** for a shortage first run followed by an `aborted` rerun, `signal` is empty, so no crash is set. `ShortageThenCrashCarriesNoCrash` would fail with the old code, which labelled it `SIGABRT`.
- **`checkHead`/`gateHead`:** `gateHead` is the only caller (verified by the earlier grep). The crash halt sits after the head-moved/dirty discard and before the push and mail. The new row mails nothing and returns a halting `Stuck`. With the defect in place the reason would be mailed and the wait would continue to the cancel, so the row would fail.
- **Merge fixer crash end:** the turn is charged before the gate, so Spent is 1. The `Terminal` result breaks the fake session loop, so it runs one turn, and the reason carries the prefix and a way forward. Without the `crashed` branch the gate would re-prompt and run two turns, so the row would fail.
- **Parent-Align row:** without the branch a fixer would spawn, so it would fail.
- **`RunnerCrashLineSpawnsNone`:** `endedOnRunnerCrash` returns true, so `runWithFixer` returns `first`. Without it a fixer would spawn, so it would fail.
- **Verifytree, loom and webster-engine tests:** each depends on the changed behaviour. The `neither line` Detail assertion correctly requires an empty string.
- **Integration steps:** the closing-line substrings match the `failedRunLabel` format.

Minor, not a finding: `TestVerifyVerb_RunnerCrashIsNoFailureToFix` waits out the real 10 s rerun delay, since the verb runs through the real `lym` with its default delay.

Tool calls:
1. `grep` for `Gate.Reason` across the non-test code, to check the other consumers of the terminal reason. My context did not show them.
