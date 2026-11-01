```yaml
lens: fix-code-correctness
outcome: reported
```

The task was to check the fixer's change set through the correctness lens. I found no findings.

- **Test with the defect in place:** the `OutputPresent` row would fail if the stop branch moved above the file check. After `Stop`, the strand is untracked. `checkLivenessTick` (`wait.go:1266-1274`) tests `allOutputFilesExist` first and returns `OutcomeDone`. With the two checks swapped, `markedStopped` would return `OutcomeDied` and `result.Outcome` would differ from the row's `OutcomeDone`.
- **No earlier path to `OutcomeDone`:** the only other `OutcomeDone` return in the poll loop (`wait.go:1096-1111`) is event-driven. It needs a Stop event, and the test's `events.jsonl` is never written. So the row exercises the liveness path.
- **`OutputMissing` row:** it behaves as before. `Stop` removes the strand, `checkLivenessTick` sees it untracked with no files and returns `OutcomeDied`, and finalize records stopped.
- **Assertions:** `RemoveStrandCalls == 1` holds in both rows, since the gate run was green. `loadRunState` is skipped for the done row, so the run directory's deletion on done cannot make it fail.
- **Comment edits:** the changes in `engine.go`, the `wait.go` file header and `docs/overview.md` touch no code, so no caller is affected. Their wording matches `wait.go:1271`.

Tool calls beyond the inherited context:
1. Bash `grep` on `wait.go` for `markedStopped|allOutputFilesExist`: my context lacked the order of the checks in `checkLivenessTick`.
2. Read `wait.go` 1255-1285: I needed the body of `checkLivenessTick`.
3. Read `wait.go` 255-325: I needed the `Wait` loop, to see whether another path returns `OutcomeDone` before the liveness tick.
4. Read `wait.go` 1085-1115: I needed to confirm that the other `OutcomeDone` return is event-driven.
