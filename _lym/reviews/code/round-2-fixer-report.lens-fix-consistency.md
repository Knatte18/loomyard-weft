```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 and F2 change set for agreement with its surroundings.

- **LOW** — `internal/gatecli/redundant_integration_test.go`, the comment above `gate`. It says "The test bound, 20 × redundantTestSecond", which repeats the literal `redundancy_test_timeout_sec: 20` from the string below it. If one is edited without the other, the comment goes stale. Say "the test bound, `redundancy_test_timeout_sec` × `redundantTestSecond`" instead.
- No other findings:
  - The only other `"redundant"` call through `Command()`, `testmap_integration_test.go:678`, targets package `solo`. It is not the endless-mutant fixture, so it has no real-second wait to fix.
  - The other test gate strings already use `redundancy_test_timeout_sec: 20`, so the new value agrees with them.
  - `runRedundant` takes the same `defaultGoBinary`, `defaultPipeWaitDelay`, `time.Minute` and `os.Environ()` that `redundantCmd` passes.
  - No `for range 16` sampling is left anywhere under `internal`.
  - `sourcescan_test.go` sets one cycle test per row, so each walk order is deterministic. The review also asked for a third row holding both cycle tests. That row would make the walk order random again, and the two single-test rows already pin each order.

Tool calls: one `grep` over `internal` for the `"redundant"` command calls, the `for range 16` pattern and the `redundancy_test_timeout_sec` config strings. My context did not hold the sibling test files.
