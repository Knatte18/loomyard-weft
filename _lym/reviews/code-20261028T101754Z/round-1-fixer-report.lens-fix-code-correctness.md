```yaml
lens: fix-code-correctness
outcome: reported
```

No findings that rise to BLOCKING. Four lower-severity ones follow.

1. **MEDIUM — `TestBoltPull_WaitsForTheBoardWriteLock`, the F7 change.** The fix swaps `time.After(200ms)` for a loop of `time.Sleep(5ms)` until a 1s deadline. That is still a fixed real-time wait, now five times longer. It still passes vacuously if `Bolt.Pull` ignored the lock but ran slowly. F7's stated premise (the repo's no-fixed-wait rule and the vacuous pass) is therefore not resolved. A real signal is needed. One option is to expose an injectable seam on `Bolt.Pull`. Another is to assert on the board write lock's state through a contender that observes the blocked acquisition.

2. **LOW — `localBeamCandidates` (`internal/fabricengine/hubpull.go`).** A non-beam pair whose origin record is unreadable is now reported as a stopped `beam` side. The record cannot be read, so the code cannot tell the pair is not a beam. The label is misleading, and the whole verb exits non-zero. Before the fix the verb returned an error, so it is not a regression. The `HubPullSide` doc comment does not mention this case.

3. **LOW — `TestPullHub_StopsABoardSideAndAPairSideWithTheirDetail` (F6).**
   - The test only asserts that the stopped detail is non-empty. It does not show which path produced the stop for the beam whose weft origin was repointed: the `!pullRes.WeftPulled` branch (`internal/fabricengine/hubpull.go:146`) or the `pullErr` branch. If `Pull` errors there, F6's "`WeftPulled`-false stop" row is still unpinned.
   - Pin the exact detail text, for example the "origin could not be reached" string.
   - `PullHub`'s list-error return still has no row.

4. **LOW — worktree order in the tests.** The moved-sides test (`b`, `t`, `b2`), the stopped-side test and the unreadable-beam test all expect `b` before `b2`. `git worktree list` gave that order when `b2` was created first (first run), so it looks name-sorted. The reason for the order is not stated anywhere. If it is really creation order, the tests become order-fragile.

Checked and holding:
- **SkipGit early return (F3):** `TestPullHub_HonorsSkipGitWithEverySideUpToDate` fails without the branch, because `Fabric.Pull` returns a zero result and the prime would show `stopped`.
- **Unreadable beam (F4):** `TestPullHub_UnreadableBeamRecordStopsOnlyThatBeam` fails against the old early return, because `PullHub` would return an error.
- **`localBeamLocations` rename:** it has no remaining callers.
- **F1 CLI test:** the cancel row correctly stops only the prime. The `RunID` check is on the waiter ticket, not the lock record, which is an acceptable substitute.
- **F8 comment rewrite:** no problem found.

Tool calls:
1. `grep` in `fabricengine/doc.go`, `fabriccli/doc.go` and all `.go` files for `localBeamLocations` and "origin record". I wanted to know whether a doc or caller still pointed at the renamed function and the old early-return behaviour; my context did not show `doc.go`. Neither does.
