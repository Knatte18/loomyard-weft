Task: check the fix's changed code through the fix-code-correctness lens (premises, callers, and whether each added test fails with the defect in place).

I traced each change by hand against the gate's passing tier-0 and tier-1 runs. I found one LOW issue and no correctness defects.

**LOW — `TestLoopIntegration_KeeperKeepsWaitingThroughTerminatingSignals` leaks the loop process when it fails** (`loop_unix_integration_test.go`)
- The loop script waits with `while [ ! -e release ]; do sleep 0.05; done`, and only the success path creates `release`.
- If the keeper's handler is missing, the `signals <- SIGTERM` send times out and `t.Fatalf` runs. The `/bin/sh` poll loop then runs forever, and the `keepLoop` goroutine stays blocked.
- A `t.Cleanup` that writes `release` would stop it. This does not affect a passing run.

What I checked, with no problem found:
- **`deadLoopStop` callers.** The new `awaitExitMark` argument is set at all three call sites: `false` for the teardown, `true` for a pid record with no envelope, and `follow.spawned == nil` after waiting.
  - `follow` is in scope at the third site.
  - A waiter that spawned the keeper skips the wait. By then its spawn has exited, and the keeper writes its mark first.
- **`awaitLoopProcessMark`.**
  - The match string `"<id> process "` does not match the spawn's `"<id> keeper process N exited"` mark. It also does not match a different id such as `dead-10`.
  - It makes at most 5 waits of 100ms each, and returns on a cancelled context.
  - A keeper start failure writes `"could not start"`, not `"process"`, so a waiter that did not spawn the keeper waits the full ~500ms there. The wait is bounded and the stop is still correct.
- **New F4 test.** Sleep 1 and 2 find no mark, and the `onSleep` at call 2 appends it. Poll 3 sees it, so there are exactly 2 waits. Without the wait, the detail lacks the mark and the test fails.
- **F9 and F8 rows.**
  - Same-producer row (status `A`, history 1, record `A`): `inFlightAt` is false, so `status_moved` is true. The old `!inFlightAt && != ""` condition would have added the "now reads A" clause, so this row fails against the old code.
  - Empty-producer row: `status_moved` is true and the cause has no clause. The `assertLoopExited` skip is needed because that helper builds its expected text from the status file's producer, which is empty there.
  - The generic `status_moved == statusMoved && !noChild` holds for every existing row.
- **F7.** The `StartFailure` row gets code 1, 0 `OnStart` runs, and the `could not start: fork/exec …` mark. The `CleanExit` row gets code 0.
- **F3.** With `DetachedLoopID` reverted to the detached flag alone, the keeper's loop id would be empty. The refusal would then print to stdout and write no envelope file, so the new subtest's empty-stdout and file assertions fail.

Further tool calls: none.
