```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. List, per behaviour the diff changes, the test that pins it.

**Per-behaviour list**

- verifyrun cancellation returns promptly, not after the child ends: pinned by `TestRun_Cancellation` (`run_integration_test.go:189`). The command sleeps 300s and the bound is 1m, so a run that waits on the child fails. Pinned.
- desktophook `AfterWindow` cuts off a hanging command: pinned by the `HangingCommandIsCutOffAtTheTimeout` row (`window_test.go`). Its `sleep 300` against the 1m bound fails if the call waits. Pinned.
- gitexec hung remote killed at the deadline: pinned by `TestRun_RemoteBounds` (`gitexec_unix_test.go:148`), with a 1m bound against the hang script's 300s sleep. Pinned.
- testcover `Run` returns within the grace while a descendant holds the pipe: pinned by the `ReturnsWithinTheGraceWhileADescendantHoldsTheOutput` row (`runner_test.go:140`). The child sleeps 5m and the row's `within` is 1m. Pinned.
- dobby fresh-start refusal after the probe grace only: pinned by the subtest at `loop_test.go:447`. `ReexecLockWait` is 10m against a 1m bound. Pinned.
- cardverify cancelled line returns after the cancel, not after the sleeper: pinned by the `TestRunner` subtest (`runner_test.go:238`). The sleeper sleeps 300s and the bound is 1m. Pinned.
- battencli `runRowCommand` ends at its timeout: pinned by `stepRunRowCommand_ReportsATimeoutAndAnExitCode`. The command sleeps 300s and the bound is 1m. Pinned.
- battencli Command-row start cost: pinned by `stepStartCost_...` with `startCostBound` raised to 2s. Pinned, but weaker; the summary admits regressions of tens of milliseconds are no longer caught.
- gatecli failed launch ends at once instead of waiting out the claim bound: pinned by the detached-run subtest. The claim bound is 10m and the limit is 1m, so a call that waits fails. Pinned.
- gatecli late runner is claimed before the cap: pinned by exit 0 alone. `SlotBusyExit` is the alternative, so the outcome tells the paths apart. Pinned.
- gatecli waiting call answers once the runner has ended instead of waiting out its cap: pinned by the 10m cap and 1m limit. Pinned.
- gatecli busy-spin ended by the watchdog's quit: pinned by `FindWatchdogQuit` returning the 1s window, plus the SIGQUIT stacks and a non-zero exit. The 120s template cannot produce a 1s window, so this is pinned.
- treadleengine lingering child does not hang past the wait delay: pinned by the two Windows-only subtests, with pings of about 2 minutes against a 1m bound. They are compiled but not run, per the summary. Pinned in design only.
- reedengine cold boot reuses the pre-started server instead of waiting out the stale-holder grace: pinned by the server-pid comparison (`ensuresession_integration_test.go`, `pidAfter != pidBefore`). A boot that waits out the grace replaces the holder, which changes the pid. Pinned.
- githubclient `resolveToken` never waits or prompts: the timing assertion is removed, so nothing pins "no wait". The seam is faked, so there is nothing to wait on. The behaviour is not pinned by any other assertion, and the removal is justified.
- githubclient `ghAuthTokenTimeout` override applies: pinned by the new deadline-left check in `TestRunGHAuthTokenSeam_HonoursGhAuthTokenTimeout`. The `elapsed < ghAuthTokenTimeout` lower bound is kept. Pinned.
- githubclient "cancelled after release ends the call": pinned by `errors.Is(err, context.Canceled)`. Waiting out the 10s deadline would give `DeadlineExceeded`. Pinned.

**Findings**

1. **LOW / design** — `internal/githubclient/githubclient_test.go`, `TestResolveToken_Chain` around line 188: the `elapsed > time.Second` check was removed, and the test's doc comment lost "without blocking or prompting". The function-doc claim was edited to match, so there is no stale claim. The "fast result, never a wait or prompt" behaviour now has no pin, but the seam is faked and nothing can wait. I raise no defect here; it is noted for the handler.

2. **LOW / design** — `internal/battencli/lifecycle_integration_test.go`, `startCostBound` (about line 1415): raising it from 100ms to 2s weakens the pin on Command-row start cost. A cost regression between 100ms and 2s now passes. The summary discloses this. Whether that dilutes the original claim is the handler's decision.

3. **LOW / consistency** — `internal/gatecli/detachfront_integration_test.go`, the late-runner test: the stand-in starts 2s late and the claim bound is now 1m with a 2m cap. A regression where the call reaches its cap is caught only after 2m. The exit code distinguishes the paths, so this is acceptable. The cost is a slow failure, not a missed one.

I found no behaviour left without a test row. The Windows-only `treadleengine` rows have not been run.

**Tool calls:** none. My context already held every file and diff I needed.
