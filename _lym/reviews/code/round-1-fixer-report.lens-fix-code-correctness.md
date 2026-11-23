```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code for the fix-code-correctness lens, using what my context already holds.

**Findings**

1. **MEDIUM — `internal/proc/processscope_linux.go`, `scopeStateEnded` / `waitForScope`:** a scope whose start job is still queued can be reported as failed.
   - The state check first runs at poll 10, about 200 ms after the busctl call returns.
   - It treats `inactive` as ended, the same as `failed`.
   - The fixer's own probe showed that a unit that does not exist, which is what a dropped unit reads as, answers `s "inactive"`.
   - A scope whose start job has been queued but has not run yet would very likely read `inactive` as well.
   - On a loaded user manager, a start that would have joined within the 5 s timeout now returns `ErrMemoryCap` after 200 ms.
   - The pane is then launched uncapped, which was right before the fix and is wrong now.
   - A sound variant would treat only `failed` as ended, or would require `inactive` on two consecutive reads.

2. **LOW — `internal/proc/processscope_state_linux_test.go`:** the new tests cover only `unitObjectPathLabel` and `scopeStateEnded`. Nothing exercises the branch in `waitForScope` that returns early. Removing the `polls%scopeStatePollEvery` block would pass every test. The F6 behaviour is therefore untested.

3. **LOW — `internal/reedcli/oomnotice_test.go`, `TestWatchPaneOOMKills_PostsANewKillOnceAndStopsWithItsContext`:** the "no duplicate post" check can pass with the defect in place.
   - If `seen` were never advanced, the 1 ms watch would re-post on every read.
   - The test calls `cancel()` right after the first line arrives and then checks `len(lines)==0`, so a duplicate may not have been posted yet.
   - `TestReportPaneOOMKills` does pin the advance deterministically, so only the wiring-level duplicate check is weak.

4. **LOW — `internal/reedcli/watchdog.go`, `enterSession` and `runWatchdogLoop`:** the watch now takes the loop's `timing`, and `NextWakeCadence` backs off only when a notice is posted. Integration runs with compressed timing, such as a ceiling equal to the cycle, will call `Engine.Status` at that short cadence for every entered session, taking the reed op lock each time. This does not affect production, which uses 5 s and 60 s. It may add op-lock contention to those tests, but I found no concrete failure.

**Checked and holding**
- `planPaneOOMNotices`: the `tmux-spawn-` row fails without the prefix check, and `wantSeen` is correct.
- `paneOOMNoticeLine`: slicing `killed[told:end]` is guarded by the length check, and the lagging-journal row falls back to the count-only line.
- `reportPaneOOMKills`: the `seen` advance after a failed post and after a successful one is pinned step by step.
- The `watchedSession` fields removed (`location`, `oomKills`) have no remaining users.
- `ws.cancel()` is now called without a nil check. Every `known` entry comes from `enterSession`, which always sets `cancel`, so this holds.
- Both `enterSession` callers in the integration tests were updated.
- The `PaneScopeUnitPrefix` rename has no other references.
- `unitObjectPathLabel` matches systemd's escaping.
- `TestLaunchStrandLocked_APaneThatCannotBeCappedStillLaunchesWithAWarning` fails with a mutant at either `panememory.go:35` or `:40`.
- `TestPaneMemoryMax` literals are correct.

**Tool calls:** none. My context already held every changed file.
