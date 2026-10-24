```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F3 change set (`activity.go`, `activity_test.go`) against its surroundings.

1. **LOW** — `internal/shuttleengine/doc.go:426`
   - The doc says the Info is logged "once it reads live again or no longer reads running". The F2 fix adds a third trigger: the record vanishes (the `!found` branch in `activity.go:85-88` now calls `Recover`).
   - A reader of `doc.go` would not expect an Info on a vanished record, and the `ReadAgentActivity` godoc (`activity.go:53-63`) doesn't say so either. Fixing it means adding "or its record is gone" to `doc.go`, or confirming that "no longer reads running" is meant to cover it.
   - The `waiterlessRuns` comment at `activity.go:109` ("whose record reads running…") and the `noteWaiterlessRun` comment ("It ends the episode when rs reads anything else") are still true.

Other twins agree with the change:
- No other file or doc quotes the old recovery text (grep found none).
- The test reads the constant `waiterlessRecovered`, and the Warn text it matches at `activity_test.go:318` is unchanged.
- Both `Recover` sites in `activity.go` (lines 86 and 93) use the same constant and key arguments.

Tool calls:
1. Grep for the recovery text and `waiterlessRecovered` across the repo. My context lacked the sibling docs that quote it.
