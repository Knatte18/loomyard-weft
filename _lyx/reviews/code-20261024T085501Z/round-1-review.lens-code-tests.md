```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds, the test that pins it, then apply test economy.

**Per behaviour**
- Waiterless running record is still skipped (no reading returned): pinned by the loop of 3 reads in `activity_test.go:299-303`, which asserts `len(got) == 0`.
- One Warn at the first skip, then Debug for each repeat, with `failed_ticks` stamped: pinned by `requireFailureEpisode(..., 2, ...)` at `activity_test.go:313`. The helper is at `fakes_test.go:95-115`.
- One Info recovery when the record reads live again, logged exactly once: pinned by the same helper (`INFO == 1`) after the `live` rewrite and read at `activity_test.go:304-307`. A broken `waiterlessRuns.Recover` call at `activity.go:92` would fail it.
- Log line names the run directory: pinned at `activity_test.go:314`.
- No-pid record logs nothing new: pinned by `run-b` (`activity_test.go:296`), which has no pid and must not appear in the log (`:318-322`). The single-recovery assertion also shows it opened no episode.
- Finished record logs nothing new: pinned by `run-c` (`activity_test.go:297`), whose outcome is done with a dead pid, under the same two assertions.
- Episode keyed separately from `unreadableFiles`: no dedicated test. It is only exercised implicitly, since a shared key would repeat the Warn on each readable read and fail the 1 Warn / 2 Debug count.
- Running record turning terminal ends the episode with one Info (the `Recover` at `activity.go:124`): none.
- Record vanishing (`!found`) leaves the episode open: none. That is code behaviour, not a stated criterion.

**Findings**
1. LOW, class `scope`, `internal/shuttleengine/activity_test.go:289-327`. The `Recover` branch of `noteWaiterlessRun` at `activity.go:124` has no test. The decision record and the `doc.go` text both state that a record "no longer reads running" ends the episode with one Info. The new subtest only ends the episode by the record reading live. Deleting `activity.go:124` would leave the whole suite green. Fix: after the live read, or in a second stretch, rewrite `run-a` with `outcome: string(OutcomeDone)` and `pid: 4343`. Read twice, then assert that a second recovery Info is not logged. Alternatively, run a separate pass where the waiterless record turns done and assert exactly one `waiterlessRecovered` Info. Do this as an extra step in the same subtest, not a new test.

**Test economy**
- No duplicate test. The existing episode subtests at `activity_test.go:259-286` cover the parse-failure episodes. The new subtest covers a different condition and message, so it duplicates none of them.
- It reuses `requireFailureEpisode` and the shared `live` and `created` fixtures, as test economy asks.
- No per-helper test of `noteWaiterlessRun` was added, and no assertion on stencil prose.

**Tool calls**
1. `Bash`: read `activity_test.go` lines 1-140 and grepped for `livePID` and `isAlive`. My context lacked the test file's fixtures (`live`, `created`) and the `isAlive` stub. It showed that pid 4343 is dead under the stub, so `run-a` reads as waiterless.
