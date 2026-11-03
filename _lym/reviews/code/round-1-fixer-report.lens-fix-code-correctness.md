```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I checked the F1 seams test and the F3 timeout row. No findings.

- **F1 test, `Compare`:** the skip file reads `s1\nt1\n`, because `SkipList` adds the skip entries and then the accepted tests. `{baseline_dir}` renders as `paths.BaselineDir`.
- **F1 test, `CompareAccepted`:** the skip file reads `s1\n` (`scope.Skip`), and `{baseline_dir}` renders as `paths.AcceptedDir`.
- **F1 test, `CaptureAccepted`:** `RunAcceptedCapture` creates the accepted dir and writes the skip file before running the command, so `accepted/skip.txt` holds `s1\n`.
- **F1 test, seam inputs:** the seams read only the step and scope, never the status. Passing `""` for the status paths is safe. `RunAcceptedCapture` takes no lock and runs no `Check`, so the unset `LockRetry` and `Check` on that seam do not matter. `RunCompare` does take the compare marker, and the compare seam sets `driftCompareLockRetry`.
- **F1 test, defect in place:** it fails if `CompareAccepted` passes `SkipList` (`s1\nt1\n`). It fails if `ProfileDir` is dropped (`dir.txt` shows the baseline dir). It fails if `CaptureAccepted` or `Compare` use the wrong skip list.
- **F3 row:** `RunAcceptedCapture` sets `"timed out after " + … + "m; log " + side.Log` when `side.TimedOut` is true. With a 300ms timeout the row ends with Passed false, a reason containing `timed out after ` and the log path, and an empty accepted dir. That satisfies every loop assertion: the log exists, no stale file, no capture record, slot freed. The row fails if the timeout branch is dropped, because the reason would then be empty or `exit …`.
- **F3 row, loop edit:** the `timeout` field and its override are consistent with the other rows, which leave `timeout` zero.

Tool calls:
1. Read `internal/driftengine/capture.go:235-345`, because my context did not hold `RunAcceptedCapture`'s lock and timeout behaviour.
