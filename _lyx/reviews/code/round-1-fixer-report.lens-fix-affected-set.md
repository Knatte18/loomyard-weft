```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix's coverage of the sets its findings name (the `usageLimited` / notice sites for F3, and the table rows for F1 and F2).

**Finding 1 — MEDIUM**
- Changed text: `internal/shuttleengine/hold.go` `holdNoticePending` (`run.covered` term) together with `wait.go:327`.
- Summary: a limited turn end read while a pause covers the run, on a `HoldDeferral` run, now skips the usage-limit case and still notifies.
- Evidence:
  - `holdNoticePending` returns false when `run.covered`, so `usageLimited` is not asked.
  - The switch falls through to `case !run.turnStartRead`, which queues `run.pendingHold`.
  - `settleDeferredHold` (`hold.go:~71-75`) later calls `notifyHeld(held)` with no `usageLimited` check.
  - `notifyHeld` drops a covered run only at the moment of the call. Once the pause lifts, the queued limited turn end is sent as a hold notice.
  - Before the fix, the `usageLimited` case ran first and swallowed it.
- This is the "queued" half of the premise, and the `covered` term wrongly assumes `covered` means no notice will ever go.
- Possible fix: drop `run.covered` from `holdNoticePending`, or check `usageLimited` in `settleDeferredHold`.

**Unchecked findings**
- F1 and F2 are single table rows. I did not re-run a search for sibling tables.

Tool calls:
1. `grep` for `usageLimited(`, `notifyHeld(` and `holdNoticePending` across `internal`: to find every site that asks the usage limit or sends a notice.
2. `sed -n 55,76p hold.go` plus `grep pendingHold wait.go`: to read the flush path, which my context lacked.
