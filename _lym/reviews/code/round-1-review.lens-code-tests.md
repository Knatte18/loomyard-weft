```yaml
lens: code-tests
outcome: reported
```

Task: map each behaviour the diff adds or changes to the test that pins it, or none.

**Per behaviour, the test that pins it, or none**

1. `TrackedFileLockPath` mirrors a file under `_lym` into the checkout's `.lym/locks`: pinned by `TestTrackedFileLockPath` (`internal/lock/tracked_test.go:27`), rows `UnderLymDir_MirroredUnderLocksDir`, `UnderAbsentLymDir_MirroredUnderItsParent`.
2. A lock reached through a linked `_lym` directory is the lock reached through the directory itself: pinned by row `ThroughLinkedLymDir_SameLockAsTheDirItself`.
3. A file under no `_lym` gets its lock beside it: pinned by row `OutsideAnyLymDir_Beside`.
4. `TrackedFileLockPath` with a relative path: none. This is the focus question. The code returns a relative lock path, and no row runs one.
5. `TrackedLockPath` joins `root/.lym/locks/name`: pinned by `TestTrackedLockPath` (`tracked_test.go:639`), but only for a plain, already-resolved root. The symlink resolution the doc comment promises has no row. `resolvedTempDir` hands in an already-resolved path, so removing `resolve(root)` would not fail it.
6. All five acquirers refuse a lock inside a tracked `_lym` directory, create no file, and accept the mirrored lock path: pinned by `TestAcquire_LockInTrackedLymDir` (`tracked_test.go:650`).
7. The refusal is skipped when a `.lym` segment appears earlier in the path: none. Nothing pins this bypass or asserts that it is intended.
8. The refusal for a lock beside a Bolt file, which the Goal's fourth bullet and `PATTERN-tracked-file-locks` ("A test refuses…") require: none. `refuseTrackedTree` only looks for the `_lym` segment. No row refuses a `board.lock` or `.swaplock` built beside a Bolt file. No scan test refuses a new `path + ".lock"` literal in a tracked tree either.
9. `BoltLockPath` creates the lock directory and returns the `.lym/locks` path:
   - Pinned only indirectly, because every moved test calls it before acquiring and would fail on a missing directory.
   - The `os.MkdirAll` failure branch has no row.
   - No row asserts the returned path sits under `<bolt>/.lym/locks`.
10. Bolt write lock, push lock and swap lock moved out of the board root: pinned by `stepSyncRetireLegacyCommitsBoardJSONAndLegacyDeletions` (`sync_integration_test.go`, new `ReadDir` loop). It asserts no `.lock` or `.swaplock` beside the board's files after a write and a sync. It runs only in the integration tier, and `lym gate test ./internal/boardengine` ran tier 0 with 0 tests.
11. The Bolt push lock is the one Sync serialises on: pinned by `stepSyncConcurrentCallSerializesOnBoardPushLock` and `TestBolt_Sync_HoldsSingleAbsorbingLockAcrossBurst`.
12. `Bolt.Pull` and `pullFirst` hold the board write lock and the git-dir push lock: pinned by `TestBoltPull_HoldsTheBoardWriteLockWhileItPulls` and `TestBolt_PullThenCommitWritten`.
13. `retryBoltPush` takes the moved write lock and push lock: pinned by `TestPushFailureClasses` (`pushclasses_integration_test.go`), which holds the moved board lock.
14. `tryRecordsLocks` on Bolt uses the moved paths: pinned by `TestTryRecordsLocks_HolderRecord` and the `Bolt push lock` row of `TestReconcileOrigin_Moves`.
15. The heddle write and render paths take the moved Bolt lock: pinned by `TestHeddleCLI` and `TestRender` (`HeldBoltLockFailsTheRender`).
16. `CommitShortname` and `CommitSeededStencils` take `BoardWriteLockPath`: none that holds the lock to prove contention. Only a build-level change is pinned.
17. `Repo.PushLockPath` is `lym-push.lock` in the git dir. `PushCoalesced` takes it, and the lock never shows in the worktree.
    - Pinned by `TestStatus_MergesUncommittedChangesBothSides_ExcludesWeftArtifacts` (the file is created there and must not show).
    - No direct `gitrepo` test asserts `PushLockPath` or that `PushCoalesced` creates that file. The existing `gitrepo` push tests are unchanged.
    - `PushLockPath` on a non-repository path (error branch): none.
18. `.gitignore` of the board loses `*.lock` and `*.swaplock`, keeps the render manifest, and is idempotent: pinned by `stepSyncGitignoreIgnoresManifestAndDropsLockPatterns` for the strip and the exact result. The no-op rewrite (`content == string(existing)` path) and a file without a trailing newline have no row. The assertions in `boardtest/sync_test.go` (`stepSyncIgnoresLockfiles`) only check the patterns are absent.
19. `seedWeftArtifactExcludes` removes the retired wildcards, adds `.weft/`, `.lym/` and the legacy names, and is idempotent: pinned by `TestCommitWeft_SeedsFabricArtifactsOnlyAndIsIdempotent`. That test pre-writes `*.lock` and `*.swaplock`, then compares the exact line set and the byte-identical re-seed. It is order-insensitive after the change.
20. A legacy lock file left in a weft checkout stays excluded, and a stray lock under `_lym` shows in `git status`: pinned by `TestCommitWeft_LockArtifactsExcludedFromStatus`.
21. Review record locks (`settle-record`, `generation-record`, patch) move to `.lym/locks/_lym/…` through `TrackedFileLockPath` in `shedadapters`: none. The tests in the diff cover only `internal/lock`. No `shedadapters` test asserts that `writeRecord`, `readRecord`, `writeSettleRecord` or `CarryRecords` leave no `.lock` file beside a record, or that the lock lands under `.lym/locks`. Reverting to `path + ".lock"` would be caught only by the runtime refusal in a test that runs under a `_lym` path. A temp-dir test whose path has no `_lym` segment would pass either way.
22. `state.WriteJSON`, `WriteFile` and `UpdateJSON` create the lock directory (`mkdirParents`):
    - Pinned only indirectly, through the callers above.
    - No `internal/state` test uses a `lockPath` whose directory is absent.
    - `UpdateJSON` is the call the `fabricengine` and `boardengine` moved sites reach.
23. `ReadJSON` with a lock directory that does not exist yet: `ReadJSON` tolerates `fs.ErrNotExist` from the read lock and reads unlocked (`state.go:109-115`). The moved sites rely on this for a checkout holding a record but no `.lym/locks` yet. No test pins it for a moved path, such as a cloned board with `board.json` and no lock directory.
24. `cmd/lym` uncontained-write allowlist entry for `junctionnames.go`: exercised by the existing scan test `uncontainedwrite_test`, which would fail without it. Pinned.
25. `PATTERN-tracked-file-locks` "(test)" claim: backed only by the `_lym`-only refusal (see 8).
26. `checklist_test.go` change: a fixture follows `shedrun.StatusLock`, no new behaviour.

**Findings**

- MEDIUM, design — no test refuses a lock path beside a Bolt file, or scans for the old `path + ".lock"` and `".swaplock"` construction, so the Goal's fourth bullet and the `(test)` marker in `PATTERN.md` are only half backed. Evidence: `internal/lock/tracked.go:869-880` (the check looks only for the `_lym` segment), `internal/lock/tracked_test.go:650-704`, and `pattern/PATTERN-tracked-file-locks.md:8`.
- MEDIUM, design — the `shedadapters` record-lock move has no test. Evidence: `internal/shedadapters/settlerecord.go:124,133,152,258`; the diff adds no `_test.go` change under `internal/shedadapters`. This is a behaviour the diff changes with no row that fails if it breaks, except under a `_lym` path.
- LOW, consistency — `TrackedLockPath`'s symlink resolution and `TrackedFileLockPath`'s relative-path handling have no row. Evidence: `internal/lock/tracked_test.go:639-648` and `:599-637`, where `resolvedTempDir` pre-resolves the root.
- LOW, consistency — the `refuseTrackedTree` early return on a `.lym` segment ahead of `_lym` has no row. Evidence: `internal/lock/tracked.go:872-873`.
- LOW, consistency — the lock-directory creation in `internal/state` and the unlocked read when the lock directory is absent have no direct test. Evidence: `internal/state/state.go:61-69` and `:106-118`.
- LOW, consistency — `gitrepo.PushLockPath` and the `PushCoalesced` lock location have no `gitrepo` test. Evidence: `internal/gitrepo/push.go:31-39`; the diff shows no `gitrepo` `_test.go` change.
- LOW, consistency — the board sync lock-location assertion lives in an integration-tier step, and `.gitignore` idempotence has no row. Evidence: `internal/boardengine/sync_integration_test.go` (the `ReadDir` loop), `internal/boardengine/sync.go:217-265`.

**Tool calls**

1. Read `internal/state/state.go` `ReadJSON` and grepped the `internal/state` tests for lock-directory creation, because my context showed only the `mkdirParents` diff and not the read path.
