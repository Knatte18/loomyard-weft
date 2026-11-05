---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: PATTERN.md:13
    summary: The durable-vs-ephemeral-state entry, which the Goal does not touch, lost the words rendered and temporary directory from its heddle exception
    fix: In PATTERN.md, make the heddle clause of the PATTERN-durable-vs-ephemeral-state line read "bar heddle's rendered files and `.heddle` at a warp worktree's anchor", which keeps "rendered" and names `.heddle` as the directory, and shorten another clause of the same line or of the new PATTERN-tracked-file-locks line if the headroom line demands it.
---

### [LOW] Untouched entry's heddle exception was reworded by the headroom commit

**Location:** PATTERN.md:13

**Issue:** The base text read "bar heddle's rendered instruction files and their `.heddle` temporary directory at a warp worktree's anchor".
The diff leaves "bar heddle's instruction files and `.heddle` at a warp worktree's anchor".
The Goal asks only for a new lock entry and does not license rewording this one.
The exception is still recognisable, and the background file `pattern/PATTERN-durable-vs-ephemeral-state.md` keeps the full wording, so this is a small weakening.
"Rendered" separates the generated files (`CLAUDE.local.md`, `AGENTS.override.md`) from heddle's source entries, and "temporary directory" says what `.heddle` is.
The `PATTERN-cwd-resolution` entry is back to its base text, and its only change is the import-list line in its background file for `lymdirs`, which the diff needs.

Settled with no finding:

- `refuseTrackedTree` is called by all five acquirers in `internal/lock/lock.go`.
  No production code outside `internal/lock` calls `flock.New`.
- A `_lym` segment refuses wherever it sits, including after a `.lym` ancestor.
  The one exemption is a `.lym` segment followed by `locks`, which returns before any later segment is judged.
  `TestAcquire_LockInTrackedTree` pins the refused `.lym/checkout/_lym/...` row and the accepted mirrored `.lym/locks/_lym/...` row.
- A Bolt directory is refused only when it holds a `.git` entry, and a `.lym` segment below it clears the Bolt state.
  The test covers the accepted pause lock, the accepted `worktrees/_bolt/lym-push.lock` with no `.git`, and the refused `_bolt/sub/board.lock`.
  `pattern/PATTERN-tracked-file-locks.md` and its `PATTERN.md` line match what the test refuses.
- `BoltDirName` is declared in `lymdirs`, a stdlib-free leaf, and `lymcwd` re-exports it.
  The `lymcwd` import allowlist, its background line and the geometry-token owner map all follow.
  `PATTERN-leaf-packages` lists no closed set for `lock` or `lymdirs` that the new import breaks.
- `TestRecordLocks_SitOutsideTheTrackedRunDirectory` fails if any of the four `shedadapters` sites reverts.
  A lock beside the record under `_lym` is refused at acquire, and the test also asserts the mirrored lock files exist.
- `ReadJSON` reads without the lock when the lock directory is absent, so the new `.lym/locks` location needs no creation on the read path.
  The writers create it, which `lockdir_test.go` pins.
- The `PushWarpRebaseFreeAt` comment ("takes no push lock") matches the code: only `bolt.go` calls `PushLockPath`.
  `PushLockPath` has a test for the git-dir place and for creation by `PushCoalesced`.
  The `TrackedLockPath` symlink-root and `TrackedFileLockPath` relative-path rows are in `tracked_test.go`.
- Existing `legacyLockFileNames` excludes stay by name while the wildcards go.
  The Goal says the excludes go, but the diff keeps these names so leftovers from earlier binaries stay out of stage-all commits.
  The code comment states the reason, and the wildcard removal the Goal asks for holds.
