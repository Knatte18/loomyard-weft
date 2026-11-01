```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff adds or changes, name the test that fails if it breaks.

**Per-behaviour list**

1. **A stop-marked handle whose strand reed no longer lists ends the run `OutcomeDied` with no error, and the record reads `stopped`.**
   - **Test:** `TestStop_WaitThatSeesTheStoppedStrandGoEndsDied` (`stop_test.go`, new).
   - **Would it fail if broken?** Yes. With the branch removed, Wait returns `errStrandNotTracked` after two checks, so the `err != nil` assertion fails.
   - **Assertions that pin the rest:** `Outcome == OutcomeDied`, the loaded record reads `runOutcomeStopped`, and `RemoveStrand` was called once.
   - **Unverified:** I made no tool calls, so I did not check that `fakeReed` repeats its last queued `StatusResult`. The test relies on that for the second status check.

2. **The `markedStopped` branch sits after the `allOutputFilesExist` check, so a marked handle with every output file present ends `OutcomeDone`.**
   - **Test:** none.
   - **Why:** the new test seeds only a missing output file.
   - `TestStop_MarkedHandleFinalizesStoppedUnlessDone` calls `finalize` directly, never `checkLivenessTick`, so it cannot catch a reordering of the two checks.
   - **Finding below.**

3. **An unmarked handle with an untracked strand still gets `errStrandNotTracked`.**
   - **Test:** the existing row "strand absent from reed's table is a mechanism failure, not died", as `summary.md:52` claims.
   - I did not open that row, so I could not confirm it still asserts this. I accept the claim as unverified.

4. **`markedStopped()` returns the mark under `recordMu`.**
   - **Test:** pinned through behaviour 1.
   - No separate test is needed. A direct test would be a per-helper test.

5. **The negative-verdict return count in `checkLivenessTick` rises from 2 to 3.**
   - **Test:** `completionsignal_enforcement_test.go`, whose audited map is updated in the diff.
   - The tripwire pins the count and fails if the new return is added or removed without the map changing.
   - The `allOutputFilesExist` call-site audit is not touched by the diff.
   - The change is licensed by the new return site.

6. **Doc comment changes** in `doc.go`, `wait.go` and `checkLivenessTick`. Static prose, exempt.

**Findings**

- **LOW, `design` (coverage gap), `internal/shuttleengine/wait.go:1267-1272`.**
  - **Behaviour:** the ordering of `allOutputFilesExist` before `markedStopped`.
  - **Gap:** no row has a stop-marked handle, an untracked strand and all output files present, expecting `OutcomeDone`. Swapping the two checks would turn finished work into a `stopped` record and no test would fail.
  - **Fix:** add that case as a second row of `TestStop_WaitThatSeesTheStoppedStrandGoEndsDied`, as a table, rather than as a new test.
  - **Evidence:** `stop_test.go` new test body, which creates `out.md` nowhere; `wait.go:1267-1272`.

- **No duplicate-test finding.** The new test duplicates neither `TestStop_MarkedHandleFinalizesStoppedUnlessDone` nor `TestRunner_StopStepRuns_StopsOnlyTheKilledStepsLiveRuns`. It is the only one that drives `Wait` through the liveness path on a marked handle.

**Tool calls:** none. The inherited context held every file I needed.
