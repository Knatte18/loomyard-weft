```yaml
lens: code-tests
outcome: reported
```

Per behaviour, the test that pins it:

1. `heldByEntry` / `readyToStart` hold on `StartRefusal`: `TestReadyToStart` row `AStartRefusalDoesNotStart`.
2. `startReady` spawns no pass for a held task: new subtest in `TestStartReady_RefusalsAndBackoff`.
3. One notice per text across ticks: same subtest.
4. Hold keeps no backoff: same subtest.
5. Changed entry renotices: same subtest.
6. Starts on the first tick after the refusal is gone: same subtest.
7. `recordRefusal` refactor leaves the refused-start path unchanged: existing refused-start subtests (not re-read).
8. `DobbyTasks` fills `StartRefusal` only for `done`: `TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord`.
9. `doneSlugRefusal` text unchanged for the hooks: existing done-slug hook tests (not re-read).
10. `BuildView` fills `WaitingTask.StartRefusal`: `finished` row in `TestBuildView/EachEntryReasonIsListed`.
11. `Ready()` false with `StartRefusal`: indirect, `done` row in `TestDobbyLines`.
12. Render `waits on a start batten refuses: <text>`: `done` row in `TestDobbyLines`.
13. Render `ready to start, last refused start: <text>`: `retry` row in `TestDobbyLines`; `due` row in `view_test.go` pins the kept refusal.
14. A held task takes no `max_runs` slot in `BuildView`: none.
15. `startReady` applies the hold only to a released task: none.
16. A held task also covered by a pause still gets record and notice: none.

Findings

- LOW, design. `internal/dobbyengine/view_test.go:32-46`. Behaviour 14 has no row. Fix: add a `BuildView` row with `MaxRuns: 1`, a held task before a ready task, expecting the ready task not to get `AtMaxRuns`.
- LOW, design. `internal/dobbyengine/starts_test.go:141`. Behaviour 15 has no row.
- NIT, design. `internal/dobbyengine/starts.go:89`. Behaviour 16 untested; a decision for the handler.

Tool calls (1):
1. grep of `view_test.go` for `Refusal`/`NextTry`/`Starts`: confirm the `due` row exists.
