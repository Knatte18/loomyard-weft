```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1–F3 fix change set through the fix-code-correctness lens.

**Finding 1: MEDIUM, `holdNoticePending` in `internal/shuttleengine/hold.go` and its `case` in `wait.go`.**
- **Summary:** a usage-limited held turn end on a covered `HoldDeferral` run is queued and notified later, which the old code suppressed.
- **Input:** `run.covered` is true, `Spec.HoldDeferral` is non-nil, and the held turn end stopped on the usage limit.
- **Old path:** `run.usageLimited(held)` matched first, so the turn end was logged and nothing was queued.
- **New path:** `holdNoticePending()` returns false because of `run.covered`, so the case is skipped. The switch reaches `case !run.turnStartRead` and sets `pendingHold`.
- **Result:** `settleDeferredHold` (`hold.go:62-76`) later calls `notifyHeld(held)`, and it never checks the usage limit.
  - If the pause has lifted by then, `notifyHeld` no longer returns early on `covered`, and the stale limit notice goes out.
  - This is the clutter the `usageLimited` case exists to prevent.
- **Evidence:** `holdNoticePending` treats `run.covered` as "no notice will go". For a deferred run that is wrong, because `covered` is transient and the queued hold is delivered after it lifts.
- **Fix:** drop `run.covered` from `holdNoticePending`. Alternatively, apply it only when `HoldDeferral == nil`, where `notifyHeld` really does discard the notice.

**Checked, no finding:**
- **Interactive and `QuietHold` runs:** with `HoldDeferral == nil`, the case is skipped and `notifyHeld` drops the notice. With `HoldDeferral` set, the hold is queued and `notifyHeld` drops it on flush. Behaviour is unchanged and no stale notice goes out.
- **`notifier == nil`:** `notifyHeld` drops the notice, so this is fine.
- **`HoldDeferral` set and `turnStartRead` true:** the case is skipped and no case matches, so nothing is queued. This is the same as before, apart from the log line.
- **F1 row (`earlierTurnEnd, limitTurnEnd, agentTurnEnd`):** `readTurnLimit` finds `agentTurnEnd` as the newest entry (not a limit), then `limitTurnEnd` as the previous one, which is a timestamp only. With no fork directory it returns false. A scan that took any limit entry in the tail would return true, so the row fails with the defect in place.
- **F2 row (sidechain limit after `agentTurnEnd`):** the entry has `isApiErrorMessage` and `rate_limit`, so without the `IsSidechain` filter the newest entry would be a limit and the result would be true. The row would fail, so it pins the filter.

**Note:** F3 has no test in `wait_test.go` for the new gating. No test would fail if `holdNoticePending` were removed. This is below the lens threshold, since the review asked for no test there.

**Further tool calls:**
1. `grep -n "pendingHold" internal/shuttleengine/*.go` — my context did not show where `pendingHold` is flushed.
2. `sed -n 55,76p internal/shuttleengine/hold.go` — I needed `settleDeferredHold` to confirm the flush calls `notifyHeld` without a limit check.
