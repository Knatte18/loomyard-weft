```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed tests.

**Per function or branch, case run, verdict**

- `verifyrun` `TestRun_Cancellation`: the command is `sleep 300` and the cancel comes on the first write. The elapsed time runs from the cancel to the return, against a one-minute bound. A run that waits on the sleep takes about 300s and fails. A run that kills at the cancel returns within about `delay` plus a kill. Correct.
- `desktophook` `TestAfterWindow` hang row: `exec sleep 300` with a 50ms timeout, bound one minute. A cut-off at the timeout passes. Waiting on the sleep takes 300s and fails. Correct.
- `gitexec` `TestRun_RemoteBounds` hung-remote subtest: the hang script already sleeps 300s, bound one minute. Correct.
- `testcover` `TestRunAlone` pipe-holding row: the child sleeps 5 minutes, `within` is 1 minute, and the check at `runner_test.go:155` is `elapsed > within`. A run that waits on the child fails. The pid file written at `testmain_test.go:115` is read at `runner_test.go:31`, so the child can be killed in cleanup. Correct.
- `dobbyengine` fresh-start refusal: `ReexecLockWait` is 10 minutes, bound one minute. A wait of 10 minutes fails the bound. Correct.
- `cardverify` cancelled line: `sleep 300`, bound one minute. The `waitUntilGone` call still checks that the sleeper is killed. Correct.
- `battencli` `runRowCommand` timeout step: `sleep 300` with a 200ms timeout, bound one minute. Correct.
- `battencli` `startCostBound`, raised from 100ms to 2s: this is an upper bound, so it can no longer fail on load alone. It still catches a multi-second wait. Correct.
- `gatecli` failed-launch call: claim bound 10m, cap 5s, launch `error`. The call must exit 1 at once. If it waited out the claim bound it would take 10 minutes and fail the one-minute check. Correct.
- `gatecli` late-runner call: cap 2m, claim bound 1m, runner 2s late. The call exits 0 once the runner is claimed. A call that reached its cap would answer `SlotBusyExit` and fail the `code != 0` check. The dropped elapsed check is covered by that exit code. Correct.
- `gatecli` waiting call whose runner gets SIGTERM: cap 10m, bound one minute. A call that waits out its cap fails. Correct.
- `gatecli` busy-spin test: the elapsed check is gone. The watchdog line `window == "1s"` and the SIGQUIT stacks prove the watchdog ended the run. The tier's 120s template cannot produce `1s`. Correct.
- `githubclient` `TestResolveToken_Chain`: the timing check is dropped and every other assertion is unchanged. Correct.
- `githubclient` `TestRunGHAuthTokenSeam_HonoursGhAuthTokenTimeout`: `resolveToken` calls the seam synchronously (`token.go:104`), so the fake's writes to `deadlineSet` and `leftAtEntry` cannot race with the test's read. `WithTimeout(10ms)` gives a time left at entry of at most 10ms, and it only shrinks under load. With the production 5s it would exceed 10ms and fail. The check is sound.
- `githubclient` `TestAuthRT_HeldCallGetsItsFullDeadlineAfterRelease`, cancelled-after-release subtest: the deadline is 10s. If the call waited out the deadline, the error would be `DeadlineExceeded` and `errors.Is(..., Canceled)` would fail. Correct.
- `reedengine` `TestEnsureSession_BootedTrueOnColdSessionFalseOnWarm`: the elapsed check is replaced by the pid comparison, which was already there. A boot that waited out the grace would replace the server and change the pid. Correct.
- `treadleengine` `TestExecGateCommand_LingeringChildDoesNotHangPastWaitDelay` (Windows only): the pings run about 119s, bound one minute, grace about 10s. Correct, not run here.

**Findings:** none.

**Tool calls**
1. `grep` in `internal/githubclient/*.go`, `internal/testcover/*_test.go` and `internal/gatecli/*_test.go`: the seam definitions, the `within` semantics and the claim-bound env var, which the manifest did not hold.
2. `Read` of `internal/githubclient/token.go` lines 86–126: to confirm `resolveToken` runs the seam synchronously, so there is no race on the fake's captured variables.
