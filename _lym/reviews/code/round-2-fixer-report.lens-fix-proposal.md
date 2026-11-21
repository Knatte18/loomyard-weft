```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals F15–F18 as the fixer carried them out.

**Findings**

- **MEDIUM** — F16, `internal/gatecli/trx_test.go`, deletion of `TestVSTestCounterReplacesAnInvocationsConsoleCounts` and `TestRemoveResultsDir`.
  - **Summary:** The deletion leaves `replaceInvocation`, `startInvocation`, `resultsFileLines` and `removeResultsDir` with no tier0 test.
  - **Evidence:** My grep over `internal/gatecli/*_test.go` finds no remaining reference to any of the four. Only the spawning test (`tierrun_spawn_test.go`) reaches them, and its first line is `//go:build !windows`, so it is tier1 only. Commit 6fc07cbe, "fix gate reach findings — add tier0 tests", added these two tests to answer the gate's reach report. The reach report lists the tier1 test as "not judged, out of process". The next gate run therefore reports these changed functions as unreached again.
  - **Case:** The reviewer's premise holds, because the tier1 test does pin the behaviour. The proposal still trades a duplicate test for a reach regression.
  - **Alternative:** Keep a minimal tier0 test that reaches them, or cover them in a `TestReadTRXResults` row, instead of deleting both tests.

- **LOW** — F17, `Config.tiersRunBy` in `internal/gateslot/tiers.go`.
  - **Summary:** It duplicates the `dropCovered` logic of `internal/gatecli/tiers.go`.
  - **Evidence:** Two separate implementations of the "covered tier is not run" rule can drift apart, and the reviewer's proposal asked for exactly this. The two currently agree: both are non-transitive and both iterate in descending order. I found no wrong result, so this is a drift risk only.

- **No findings** for F15 and F18.
  - **F15:** An ungated writer on a hub with an absent `gate.yaml` now builds its table. The `wiring_test.go` row was updated for that, and the run fails at its own verify, which matches the old premise.
  - **F18:** The literal result words match what `vstestResultsArgs` returns. This is the intended tautology fix, and the literals do not reference `trxLogFilePrefix`, so the test would catch a helper change.

**Calls made**
1. `grep` over `internal/gatecli/*_test.go` for the four function names, plus the `tiergate.Require` and build-tag check: my context did not show whether any other tier0 test reaches them.
