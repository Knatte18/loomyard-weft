```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the fix introduced (refusals, fallbacks, next steps, criteria) can be reached from the state it names and makes progress.

**[MEDIUM] D1 / D9 / D10: the failed-probe path says "unplaced and unrecorded", but the window is already recorded.**
- Changed text in D1: "When the probe fails, the window stays where KWin opened it, unplaced and unrecorded". The D1 acceptance criterion repeats it: "a query report whose probe fails leaves the `--separate` window unplaced and unrecorded".
- Evidence: D10 says "The child records the window (D9), waits for it to appear in KWin…". D9 says "A placement first records the new window's process id and title, before any wait". The probe runs in the first query script, which is the window wait's query in D10. So the pre-wait write has already happened before the probe can fail, and "unrecorded" cannot hold.
- What it breaks: the entry is left in the state D9 calls unplaced. D8's parent wait then has no way to end for that parent's children. Each child waits the full 180 s on every open, then falls back to the current desktop, with `window_command` held back for that long. The same dead end follows a parent child that ends on a window timeout or a lock timeout: its entry stays unplaced too. D8's bound covers only "a parent whose window lym never recorded".

**[MEDIUM] Constraints, the "Not yet observed" bullet: no one is named to carry out the step.**
- Changed text: "Each is run on Neve and recorded here, with its command and output, before code relies on it".
- Evidence: every listed fact needs a KWin script loaded through `qdbus6` on the operator's live session bus. Building a grid and calling `manage` on a non-current desktop also rearranges the operator's real windows. This session's rules refuse session-bus access and send it to the operator. A plan writer or implementer working under the same rules cannot carry the step out.
- What it breaks: the step names no actor, such as the operator, or the contract test the operator runs in the real-desktop tier. As written it leads nowhere, and the fix should name who runs it.

**[LOW] D10: the 120-second bound on `lock.AcquireWriteLock` has no way to end the call.**
- Changed text: "It takes it through `lock.AcquireWriteLock`, a blocking acquire that wakes only when the lock frees, and ends the wait after 120 seconds".
- Evidence: `internal/lock/lock.go:31-40`. `AcquireWriteLock` calls `fl.Lock()` and takes no context or deadline. The only way to end the wait is to abandon the call and exit the process, and the text never says that.
- What it breaks: D10's timeout path and D12's "after a lock timeout" path are reachable only if the plan knows to exit while the acquire is still blocked. The fix should say the child exits with the acquire still pending, and that exiting drops the attempt.

No further tool calls made.
