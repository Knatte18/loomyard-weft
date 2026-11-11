```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (`log: goFailure` added to the "a passing verify runs no rerun" row in `internal/landingshed/verifygate_test.go`) for consistency with its surroundings.

No findings.

- The new field matches the `log` field's comment, "every verify call's log". `writeVerifyLog` and the fake's `writeLog` (line 83) write a log for a `StatusPassed` result too, so `first.Log` is now non-empty and only the `first.Status != verifytree.StatusFailed` check at `verifygate.go:296` keeps the row from rerunning.
- The row's `wantCommands` (one call) and `wantReasonLog` (0, a pass) are unchanged and still agree with the other rows.
- `wantFlaky` stays empty, so the `pass.log != logOf(1)` check is not reached.
- Nothing else cites this row. The test doc comment ("which failures rerun") still fits.
- `lym gate test ./internal/landingshed` passed after the change (811 passed, 0 failed).

Further tool calls: none, because my context already held everything the lens needed.
