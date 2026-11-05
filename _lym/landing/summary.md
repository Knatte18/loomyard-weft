# Lock files live in .lym, never beside a tracked file in _lym or Bolt

Locks for tracked files used to sit beside the files they guard: `board.lock`, `board.push.lock`, `board.json.swaplock` and `.gitrepo-push.lock` in the Bolt checkout, and `generation-record.json.lock`, `settle-record.json.lock` and `settle-record.patch.lock` under a run's `_lym/reviews/` directory.
Only the `*.lock` and `*.swaplock` lines in the weft repo's `info/exclude` and the Bolt `.gitignore` kept them out of git.
This change moves every such lock under a never-tracked directory and drops those wildcard lines, so a stray lock in a tracked tree now shows in `git status`.

## What changes

- **One helper.** `internal/lock` gains `TrackedLockPath(root, name)`, which returns `root/.lym/locks/<name>`, and `TrackedFileLockPath(path, suffix)`.
  For a file under a `_lym` directory, `TrackedFileLockPath` mirrors the file's path under `.lym/locks/_lym/…` in the checkout that holds that `_lym`, for example `<weft>/.lym/locks/_lym/reviews/code/generation-record.json.lock`.
  It follows symlinks, so a record reached through a warp worktree's linked `_lym` gets the same lock as one reached through the weft.
  A path under no `_lym` keeps its lock beside it.
  `lymdirs.LocksDirName` names the `locks` directory.
- **The check.** Every acquire in `internal/lock` refuses a lock path that sits inside a `_lym` directory and outside any `.lym` one, with an error that names `lock.TrackedFileLockPath` as the way forward.
  For the Bolt checkout, the board sync test asserts that the board root holds no `*.lock` or `*.swaplock` file after a write and a sync.
- **Bolt.** `fabricengine.BoltLockPath(boltDir, name)` places a lock under the Bolt checkout's `.lym/locks/` and creates the directory.
  The new `BoardPushLockFile` constant names the push lock.
  `BoardWriteLockPath(hub)` now also returns an error.
  Every board write-lock and push-lock site moves to `BoltLockPath`: `Bolt`, `retryBoltPush`, the origin reconcile's lock probe, the stencil and shortname commits, `boardengine`'s critical section and sync, and heddle's render and save.
  The `board.json` swap lock moves too, through `boardengine.swapLockPath`.
- **Push lock.** `gitrepo`'s single-pusher lock moves into the checkout's own git dir under the new name `lym-push.lock`, through the new `Repo.PushLockPath()`.
  It is a per-repository lock that any repo can hold, the user's warp included, and the git dir is never tracked.
  `Bolt.pullFirst`, `retryBoltPush` and the reconcile lock probe take the same path.
- **Review records.** `shedadapters`' settle, patch and generation records take their locks through `TrackedFileLockPath`.
- **State writers.** `state.WriteJSON`, `WriteFile` and `UpdateJSON` now create the lock's directory as well as the record's, because the two can sit in different trees.
  Readers still create nothing: when the lock directory is absent, a read runs without the lock.
- **Excludes.** `seedWeftArtifactExcludes` removes the `*.lock` and `*.swaplock` lines and no longer seeds `.gitrepo-push.lock` as a fabric artifact.
  It seeds `.weft/`, `.lym/` and the legacy lock file names, each by its exact name.
  `boardengine.ensureBoardIgnores` replaces `ensureLockfilesIgnored`: it keeps the render manifest in the Bolt `.gitignore` and removes the two lock patterns.

## Design decisions

- **Legacy lock files are excluded by name, not deleted.** Existing hubs hold leftover lock files in the Bolt root and in running task wefts.
  Once the wildcards go, the Bolt's stage-all commit and a weft's `_lym` pathspec commit would sweep those files in.
  Deleting them while an older binary might still hold one would split that binary's mutual exclusion, so each legacy name stays excluded instead.
  No code writes those names any more, and the acquire guard refuses new ones under `_lym`.
  The leftover files are inert.
  A task weft's leftovers go when its pair is torn down, but the Bolt root's stay until removed by hand.
- **The push lock goes to the git dir, not `.lym/locks`.** `gitrepo` works on any repository, and only the git dir is untracked in every one of them without an exclude.
- **The guard returns an error.** A lock under `_lym` is a programmer error, so it fails loudly with a way forward rather than silently writing into a tracked tree.
- `PATTERN.md` gains `PATTERN-tracked-file-locks`.
  The durable-vs-ephemeral and fabric-git background files now say that locks sit under `.lym/locks/` and that the Bolt `.gitignore` holds only the manifest pattern.

## Open risks

- During an upgrade, old and new binaries take board locks at different paths, so they do not exclude each other until every process runs the new binary.
- On Windows, `_lym` in a warp worktree is a junction.
  If `filepath.EvalSymlinks` does not follow it, a record reached through the warp gets a different lock file from the same record reached through the weft.
  The run's own review records are reached through one path, so in practice they keep a single lock.

## Tests

- `internal/lock/tracked_test.go` covers both helpers: the mirrored path, the path reached through a linked `_lym`, an absent `_lym`, and a path outside any `_lym`.
  It also checks that all five acquire functions refuse a lock under `_lym` and accept one under `.lym/locks`.
- The fabricengine exclude tests now assert that the wildcard lines are removed, that the legacy names are seeded, that a legacy push lock stays hidden, and that a stray `_lym/*.lock` shows in `git status`.
- The board sync scenario asserts that the retired `.gitignore` lines are removed, that the push lock sits in the lock directory, and that no lock file is left at the board root.
- Tests that hold the board, Bolt or push lock now build the path through `BoltLockPath` or `PushLockPath`.
