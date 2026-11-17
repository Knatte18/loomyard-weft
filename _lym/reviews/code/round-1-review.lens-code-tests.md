```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Behaviour → pinning test**

1. The sweep removes a dead run directory holding a read-only directory with an unreadable one under it.
   - Pinned by `internal/lymtemp/sweep_test.go` `TestSweep` row "dead owner holding a read-only and an unreadable directory" (kind `lockedTree`).
   - If `makeTreeOwnerWritable` were dropped, `os.RemoveAll` would fail with EACCES on `read-only/unreadable/left.txt` and the `applied.Failures` check would fail.
   - Run as root, the row passes but pins nothing; it does not fail there. Not a defect, since every row tolerates root.
2. A dry run removes nothing and lists the locked tree.
   - Pinned by the same row's `dry` assertions: the candidates equal `[path]`, and `Lstat(path)` still succeeds.
3. "A link is never followed": the walk skips a link inside the tree that points at a directory outside it.
   - **None.** The "prefixed link" row covers only a top-level link, which `ownerDead` rejects before `remove` is called.
   - Nothing puts a link inside a dead run directory and asserts that its target's mode is unchanged.
4. "An entry that vanishes mid-walk is skipped": the `fs.ErrNotExist` filter on the recursive call and on `removeTreeMakingWritable`'s own call.
   - **None** that is deterministic.
   - "two concurrent sweeps both succeed" uses directories that hold only a lock file, so the recursion's not-exist branch is never reached.
   - "a path gone between attempts counts as removed" replaces `root.remove`, so it bypasses `removeTreeMakingWritable`.
5. A chmod or `ReadDir` failure in `makeTreeOwnerWritable` is returned and aborts the removal, reported once as a failure.
   - **None.** The rows "another error is reported after one attempt" and "not-empty on a path inside is another error" both replace `root.remove`.
   - So the error return of `removeTreeMakingWritable` is never exercised through the default `remove`.
6. `openRoot` now wires `remove` to `removeTreeMakingWritable`.
   - Pinned indirectly by the `lockedTree` row. `newTestRoot` presumably goes through `openRoot`; I did not check that.
7. `permkit.Chmod` sets the mode and restores the original bits, not a fixed `0o755`.
   - Pinned by `internal/testkit/permkit/permkit_test.go` `TestChmod_SetsTheModeAndRestoresItAtCleanup`: it sets `0o750` before and checks `0o555` inside the subtest and `0o750` after.
8. `permkit.Chmod`'s restore drops its error when the test removed the path.
   - Pinned indirectly by the `lockedTree` row: the sweep removes both chmodded paths, and the restore's error would otherwise fail the row.
   - No permkit test of its own covers this.
9. `permkit.Chmod` fails the test when the stat or chmod fails.
   - None. This is a kit's own setup `Fatalf`, which needs no test.
10. Converted call sites.
    - Each is a behaviour-preserving swap, pinned by its own existing test unchanged: battencli, burlerengine, configcli, configengine, fabricengine, frictionengine, gatecli, impactset, logger, machinecli, reedcli, reedengine, shedadapters, shedrun, shuttleengine, vscode, websterengine and forkaudit.
11. Test economy: `internal/logger.TestSweep_DeleteFailureTolerance` is reported contained by `TestSweep_AgeBound`.
    - `TestSweep_AgeBound` asserts only that the old file is absent and the recent file exists.
    - `TestSweep_DeleteFailureTolerance` asserts that `Sweep` returns nil when an unlink fails and the stale file stays (`internal/logger/retention_test.go`, its function body).
    - Its assertions pin a failure path that `TestSweep_AgeBound` never checks, so it should be kept and marked.

**Findings**

- **MEDIUM**, design (test coverage)
  - Location: `internal/lymtemp/sweep_test.go`, `TestSweep` rows.
  - Summary: the safety claim that "a link is never followed" in `makeTreeOwnerWritable` (`internal/lymtemp/sweep.go`, the `if !entry.IsDir() { continue }` loop) has no row.
  - Fix: add a link inside a dead run directory that points at an outside directory with a restricted mode, and assert after the apply that the target's mode is unchanged.
  - Evidence: `internal/lymtemp/sweep_test.go`, the "prefixed link" row and the `lockedTree` case.
- **LOW**, design (test coverage)
  - Location: `internal/lymtemp/sweep.go`, `removeTreeMakingWritable` and `makeTreeOwnerWritable` error returns.
  - Summary: no test drives a chmod or `ReadDir` failure through the default `remove`, so whether that failure is reported once and the run directory left in place is unpinned.
  - Evidence: the rows "another error is reported after one attempt" and "not-empty on a path inside is another error" in `sweep_test.go` both replace `root.remove`.
- **LOW**, design (test coverage)
  - Location: `internal/lymtemp/sweep.go`, the `fs.ErrNotExist` filters in `makeTreeOwnerWritable` and `removeTreeMakingWritable`.
  - Summary: "an entry that vanishes mid-walk is skipped" has no deterministic row.
  - Evidence: the rows "two concurrent sweeps both succeed" (directories with no subdirectories) and "a path gone between attempts counts as removed" (replaced `remove`) in `sweep_test.go`.
- **LOW**, consistency (test economy)
  - Location: `internal/logger/retention_test.go`, `TestSweep_DeleteFailureTolerance`.
  - Summary: the redundancy report lists it as contained by `TestSweep_AgeBound`, but it pins the nil return and the kept stale file on a failed unlink, which `TestSweep_AgeBound` never asserts.
  - Fix: mark it `//testtiming:keep pins that Sweep returns nil and leaves the file when an unlink fails` directly above its `func Test…` line.
  - Evidence: both function bodies in `internal/logger/retention_test.go`; there is no existing `testtiming:keep` there (the grep returned none).

Tool calls: none. The inherited context held every file and test this lens needed.
