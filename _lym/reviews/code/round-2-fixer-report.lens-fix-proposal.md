```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 and F2 proposals as the fixer carried them out.

No findings.

- F1, `Since` read-back: with the advance and the rewritten `Modified` before the restart, a dobby that lost `Since` would stamp a new driver-dead key and send a second notice, failing the `len(got) != 1` assertion. The row is not vacuous.
- F1, `APIRevives` read-back: a dobby that dropped it would count 0→1 on episode 3 and send no notice, so `ARestartBetweenThemKeepsTheCount` fails as intended.
- F2: `durationFromCount` matches the old switch for every input. Int64 and negative-float rows are missing, but they take the same `count > 0` branches as the rows present.
- F2, residual gap (not a finding): the row lookup in `runShedDuration` and the unit at each `hubDobbyDeps` call site remain pinned only by the integration test's read of the shipped 30. The reviewer's proposal did not ask for more than the helper.

Tool calls (2, both read-only):
1. `grep` over `internal/dobbyengine/*.go` for `APIRevives`, `.Since` and `RunNoticeState`.
2. `sed` on `internal/dobbyengine/runnotice.go` lines 175-222 and 320-350.
