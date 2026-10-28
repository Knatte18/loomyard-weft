```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F8).

**Findings**

1. **MEDIUM** — `TestBoltPull_WaitsForTheBoardWriteLock`, the F7 replacement loop (`deadline := time.Now().Add(time.Second)` with `time.Sleep(5 * time.Millisecond)`).
   - It is still a fixed real-time wait, now a full second per run, so the testing rule the finding cited is still broken.
   - A pull that ignored the lock but took longer than 1s on a loaded gate host still passes. The vacuous pass is unchanged.
   - Evidence: the `done` channel is the only signal the loop checks, and nothing observes the pull being blocked.
   - The reviewer's other option was an injectable wait. A real signal would be a hook that fires when the pull reaches `AcquireWriteLock`.

2. **LOW** — `internal/fabricengine/doc.go:1226` ("A beam whose code branch has no upstream is up to date... and a pair whose records side could not reach origin is stopped") and `:1225`, "the pull-first path".
   - The doc was not updated for the fix. It never says that `SkipGit` leaves every side up to date.
   - It never says that a beam whose origin record cannot be read is reported stopped, and that the sides after it still run.
   - Both are new observable `PullHub` behavior from F3 and F4, and the project rule is that module docs land in the same commit.

3. **LOW** — `localBeamCandidates`, the F4 change.
   - A corrupt origin record is read for every non-main worktree before the beam mark is known. A corrupt record on a task pair (not a beam) is now reported as `Kind: HubPullBeam ... stopped`, so the verb fails and the side is mislabelled.
   - Before the fix this aborted everything, so it is no worse. But the proposal's premise that the failing slug is a beam does not hold for the failing read.
   - It is a defensible trade-off, and a note would cover it.

4. **LOW** — the F3 early return in `pullPairSide`.
   - It runs before `Open` and before `lockPair`, so with `SkipGit` set no side takes the parent lock and the prime's code is not touched. This is consistent with Bolt.
   - The detail text "git is skipped, so nothing was pulled" is attached to `up_to_date`, and the CLI envelope shows `up_to_date` for a pull that did nothing. No case where the proposal is wrong; `SkipGit` is an env override.

**Checked, no finding**
- F1: the same-PID holder is distinguished from the pull by slug (`SameProcess` needs `SameRun`), so the wait is real.
- F5: restoring the original help text matches `fabricengine/doc.go`.
- F2, F6 and F8 hold.

**Calls**
1. `grep -n -i "hub pull\|PullHub"` on `internal/fabricengine/doc.go` and `internal/fabriccli/doc.go`: my context did not show whether the module docs mention the changed `PullHub` behavior.
2. `sed -n 1222,1236p internal/fabricengine/doc.go`: to read the hub-pull doc text itself, which my context lacked.
