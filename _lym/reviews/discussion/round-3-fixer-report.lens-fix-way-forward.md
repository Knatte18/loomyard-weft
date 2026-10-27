```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the fix added can be reached and makes progress.

**[LOW] The kept `State.Tidy` rule needs a second marker test, and Decision 1 never names it**

- **Changed text:** `_lym/discussion/decision-record.md`, Decision 1: "The kept `State.Tidy`, nil included, is the one read by the last tick that ended with the marker present; a tick that ends with the marker gone ... never replaces the kept value."
- **Issue:** Decision 1 only tests the marker "at the top of every loop iteration ... before the owner read and the tick". Nothing tests it when a tick ends. The top check of the next iteration cannot stand in for one, for two reasons:
  - A tick can end with the marker present and have the worktree removed during the sleep that follows. The next top check then finds the marker gone. It cannot tell this tick apart from one that ended with the marker already gone.
  - So an implementation that only has the stated check has two choices. It can drop that valid read, which leaves the kept value two ticks stale and misses a tidy spawned in the last tick before removal. Or it can commit every read, which lets the recreated empty `State` replace the kept value and stops nothing.
- **Fix:** Decision 1 should state a marker test when each tick ends, or right after the tick's state reads. That test commits the read only when the marker is present. Without it the rule cannot be carried out as written.

The new rows are reachable:
- **The file-event acceptance criterion:** `loop(ctx, events <-chan struct{})` already takes the events channel as a parameter. A test can wake the loop without a probe tick.
- **The no-ticks notice criterion:** both its refused re-exec and wiring-failure paths come out of runDobby/`Run` reading `state.json`, which a test can seed with a pid and no start ticks.

No further tool calls made.
