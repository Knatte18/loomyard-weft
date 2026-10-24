```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3 as the fixer carried them out.

**Finding 1: MEDIUM** — `holdNoticePending` in `internal/shuttleengine/hold.go`, used in the `wait.go` switch (`case run.holdNoticePending() && run.usageLimited(held)`).
- **Summary:** the F3 proposal brings back the stale notice for a usage-limited turn end when a pause covers the run and `HoldDeferral` is set.
- **Evidence:**
  - `holdNoticePending` returns false whenever `run.covered` is true.
  - The limited case is then skipped, and the switch falls to `case !run.turnStartRead`, which queues `run.pendingHold` (`wait.go:333`).
  - `notifyHeld` skips only while `run.covered` is true (`hold.go:153`), and `covered` is transient. After the pause lifts, `settleDeferredHold` (`hold.go:66-75`) sends the queued notice.
  - That is the stale notice for a limited turn end that the branch exists to suppress. Before the fix, `usageLimited` came first and the turn end was never queued.
- **Case:** a deferred-hold run reads a limit-stopped turn end while a pause covers it. The pause lifts, `settleDeferredHold` fires, and the parent gets the notice.
- **Cause:** the F3 `fix` key says to apply `usageLimited` only where a notice would otherwise be sent or queued. `covered` is not a permanent skip like `Interactive`, `QuietHold` or a nil notifier, because a queued hold can still be sent after the pause lifts.
- **Fix:** drop `run.covered` from `holdNoticePending`. Interactive, `QuietHold` and nil-notifier runs never notify, so queuing there is harmless.
- **No test:** none of the new tests exercises this path.

**Finding 2: LOW** — `holdNoticePending` duplicates the skip conditions of `notifyHeld` (`hold.go:153`).
- **Summary:** the twin condition lists can drift apart.
- **Evidence:** the two lists agree today except for `held.offset <= NotifiedOffset`, which is not mirrored. A change to one list silently makes the other wrong. This does not break anything now.

**F1 and F2 rows:** no findings. Each fails with its defect in place.
- F1 transcript `[earlier, limit, agent]`: a scan that takes any limit entry would return true, and the expected value is false.
- F2 transcript `[earlier, agent, sidechainLimit]`: dropping the `IsSidechain` filter makes the sidechain limit entry the newest turn end and returns true, and the expected value is false.

Further tool calls:
1. Read `hold.go:40-76` and grepped `pendingHold`, to confirm that `settleDeferredHold` calls `notifyHeld` after the pause lifts and where `pendingHold` is set. My context did not show that function.
