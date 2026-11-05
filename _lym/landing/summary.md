# Lock files leave tracked trees: locks live under .lym/locks

Lock files of tracked files used to sit beside them, in `_lym/` and the Bolt checkout, hidden only by `*.lock` and `*.swaplock` lines in `.git/info/exclude`.
Locks are machine-local, so they now live under the owning checkout's `.lym/locks/`, and the two wildcard excludes are gone: a stray lock in a tracked tree shows in `git status` instead of hiding.

## What changes

- `internal/lock` gains two path helpers.
  `TrackedFileLockPath(path, suffix)` puts the lock of a `_lym` file under `.lym/locks/_lym/…`, mirroring the file's place under the nearest `_lym` directory.
  `TrackedLockPath(root, name)` puts a named lock of a checkout (the Bolt board and swap locks) under `.lym/locks/`.
  Both resolve symlinks, so a lock reached through a linked `_lym` directory is the lock reached through the directory itself.
- Every `path + ".lock"` and `.swaplock` site over a `_lym` or Bolt file moves to the helpers: `boardengine`, `fabricengine`, `heddlecli`/`heddleengine`, `shedadapters` (settle and review records) and others.
- `state.WriteJSON`, `WriteFile` and `UpdateJSON` create the lock's directory as well as the file's, since the two now sit in different trees.
- `gitrepo`'s push lock moves into the checkout's own git dir (`lym-push.lock`, from the new `Repo.PushLockPath`), where no commit can stage it.
- `lymdirs` declares `BoltDirName` and `LocksDirName`; the geometry-literal enforcement test covers the Bolt literal.
- `PATTERN-tracked-file-locks.md` records the rule, and the related patterns point to it.

## Enforcement

Every `internal/lock` acquire (write, read, try, within) refuses a lock path inside a tracked `_lym` directory or a tracked directory of a Bolt checkout, with a refusal naming the helpers.
The `.lym/locks` directory is exempt, and a `_bolt`-named directory without a `.git` entry (such as one in a git dir) is not treated as a Bolt checkout.
This makes the check a runtime refusal, so it also catches a hand-built lock path in tests and production, not only a lint pass.

## Design decisions to review

- Excludes: `seedWeftArtifactExcludes` now removes the `*.lock` and `*.swaplock` lines from an existing `.git/info/exclude`.
  It keeps excluding, by exact name, the lock files earlier binaries left (`board.lock`, `board.push.lock`, `board.json.swaplock`, `tasks.json.swaplock`, `notes.json.swaplock`, `.gitrepo-push.lock` and the three review-record locks), so a leftover never enters a stage-all board commit or a records commit.
  Nothing writes these names any more.
- A file under no `_lym` directory still gets its lock beside it from `TrackedFileLockPath`; only tracked trees are covered.
- The push lock sits in the git dir rather than under `.lym/locks`, because a caller that must keep a coalesced push off the checkout takes the same file via `PushLockPath`.

## To check by hand

Nothing outside the test suite was run by hand, and no regression compare ran for this task.
Check on a hub with running task wefts and an existing Bolt checkout that:

- after upgrade, `git status` in a task weft and in Bolt is clean and old lock files do not reappear;
- leftover legacy lock files stay out of board and records commits;
- concurrent board writes and `lym fabric` pushes still serialise on the new lock locations.
