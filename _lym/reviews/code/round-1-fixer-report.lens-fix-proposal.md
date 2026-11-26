```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that were carried out as written (F1, F2, F4, F5, F6, F8–F12).

**fix-proposal findings**

1. **MEDIUM**, `main.go` package doc and the F2 `signal.NotifyContext(..., os.Interrupt, syscall.SIGTERM)` call.
   - **Problem:** the proposed signal set leaves out SIGHUP, yet the doc says the probe deletes every folder it makes before it exits and names only SIGKILL as leaving the `-dir` folders behind.
   - **Case:** the operator runs a probe over SSH or in a terminal window that closes or disconnects. SIGHUP takes Go's default action and ends the process, so no `defer` and no `lymtemp.Remove` runs. The `machinecost-*` folders stay in the `-dir` folders, which F2's own premise says nothing sweeps.
   - **Evidence:** the changed doc reads "It deletes every one of them before it exits, also when an interrupt or SIGTERM ends the run early. // A SIGKILL leaves the folders inside the -dir folders behind."
   - **Change:** add `syscall.SIGHUP` to the `NotifyContext` call and name it in the doc.

I found no concrete breaking case for the other proposals. F7 was changed beyond its proposal, so I did not challenge it. I did not run F10's claim that the test fails without `-B`.

I made no tool calls.
