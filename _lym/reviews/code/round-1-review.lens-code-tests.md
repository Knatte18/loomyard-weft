```yaml
lens: code-tests
outcome: reported
```

Task restated: lens code-tests. For each behaviour the diff adds or changes, I name the test that pins it, or none. I made no tool calls and judged from the diff in context.

**Per behaviour: pinning test**

1. `entryEnded` (done, or gone from the board): pinned by `TestEntryEnded` in `internal/battencli/dobbyreading_test.go`, with rows for done, gone, run status and no status.
2. `DobbyRuns` setting `RunView.EntryEnded` from `entryOf`/`onBoard`: **none**. Only the helper is tested. If the assignment were dropped or wired to the wrong value, no row would fail.
3. `DobbyRuns` setting `RunView.PairPresent` from `markerExists(pair.WorktreePath())`: **none**.
4. `slotsTaken` and `runsInFlight` leaving out `EntryEnded` runs: pinned by the `a run whose board entry ended takes no slot whatever batten's state` subtests in `internal/dobbyengine/starts_test.go`, for both failed and running states.
5. `strandedEnded` conditions: pinned by `TestStrandedEnded`, with a row per condition (state, entry open, pair gone, in flight, driver lock, cancel marker, covered, owned elsewhere).
6. `tearDownEnded` start-and-read cycle, in `internal/dobbyengine/ended_test.go`:
   - Removed pair logs once, and no teardown runs once the pair is gone: pinned by `TestTearDownEnded`, subtest `a removed pair logs once…`.
   - A removed run waits one probe interval: pinned by the same subtest, through the call counts after the first two ticks.
   - A failing teardown logs once and backs off by one then two intervals: pinned by `a failing teardown logs once…`.
   - A teardown in flight is not doubled and keeps the loop busy: pinned by `a teardown in flight is not doubled…`.
   - A slug left out of the result fails as unreported: pinned by `a result that leaves the run out…`.
7. The one-hour backoff ceiling (`endedTeardownCeiling`, `min(...)`): **none**. The failing test stops at the second doubling.
8. A run no longer stranded forgets its wait (the `delete(d.endedRetries, …)` loop): **none**. The removed-pair subtest passes whether or not the entry is deleted.
9. A nil `Deps.TearDownEnded` tears nothing down: **none**.
10. The binary check held while `d.teardown != nil`, in `loop.go`: **none**.
11. Several stranded runs in one `TearDownEnded` call, each read by its own result: **none**. Every test uses a single slug "x".
12. `endedTeardownResults` and `endedTeardownResult` in `internal/beamcli/dobbydeps.go`: pinned by `TestEndedTeardownResults`, with rows for removed, busy, failed, another run's removal, active, unreachable and reconcile error.
13. The `tearDownEnded` goroutine in `internal/beamcli/dobbydeps.go` (`pairteardown.New` failure path, mutation logging, channel delivery): **none**. Only the report mapping is tested.

**Findings**

- F-A, LOW, design, `internal/battencli/dobbyreads.go:165-171` and `:190`: the two `DobbyRuns` assignments, `EntryEnded` and `PairPresent`, are the behaviour that frees the slot in production. No row would fail if either were dropped or miswired. I did not look for an existing `DobbyRuns` test, and the manifest holds none. Fix: add an assertion to an existing `DobbyRuns` reading test (not a new test) for a done entry with its worktree present, and one for an entry gone from the board.
- F-B, LOW, design, `internal/dobbyengine/ended.go:99-118`: the one-hour ceiling on the retry wait has no row. Fix: extend the failing-teardown subtest's fake with enough results to reach the ceiling, or assert `endedRetries[slug].interval` stays at or below `endedTeardownCeiling`.
- F-C, LOW, design, `internal/dobbyengine/ended.go:73-77`: the forgetting of a wait for a run that is no longer stranded has no assertion. Fix: in the removed-pair subtest, assert `h.d.endedRetries` has no "x" after the `PairPresent=false` ticks.
- F-D, LOW, design, `internal/dobbyengine/loop.go:563-572` and `ended.go:44-46`: no test pins the binary check being skipped while a teardown is in flight, or a nil `TearDownEnded` doing nothing. Fix: add a row to `TestTearDownEnded` that counts `CheckBinary` calls while a never-delivering teardown is pending, and one for a nil seam.
- F-E, NIT, design, `internal/dobbyengine/ended_test.go`: all subtests use a single slug. Fix: give one subtest two stranded runs with a mixed nil and error result, and assert each run's log line and wait.

Test economy: I found no duplicated coverage. `TestEntryEnded` is unique per the redundancy report. The `starts_test.go` row covers slot counting and not the stranded logic.

Tool calls: none were used.
