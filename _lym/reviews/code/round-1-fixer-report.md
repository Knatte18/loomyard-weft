# Fixer-report, round 1

F1 is `decision`-class: it went to the parent, whose recorded decision accepts both departures. It is left out of this report.

## Changes

Both files are in `internal/testkit/tmuxkit/`. The tmuxkit gate passes, and the changes are committed in `09f144b5` and a follow-up wording commit.
The fixes for F2 to F6 share one commit, because they edit the same two functions and one test table.

- **F2**: `runCheckingGitEntries` in `tmuxkit.go` takes the snapshot, calls the run function, and checks. `Main` passes `m.Run` on the Windows and Linux paths. `TestRunCheckingGitEntries` covers four cases: a run that creates no entry keeps its code, a run that creates one returns 1, an incoming failing code still returns 1, and an entry present before the run keeps the code.
- **F3**: `checkNewGitEntries` reports each outcome separately through `removeStrayGitEntry`. It says not a directory for a file or symlink, removed for an empty directory, and already removed when `os.Remove` returns `fs.ErrNotExist`. It says not an empty directory for a populated one, and prints the `os.Remove` error otherwise. A follow-up commit made the last message neutral ("could not remove %s"), since it is also reached when the directory cannot be read.
  - Fixed differently: the review proposed saying already removed when `Lstat` returns `fs.ErrNotExist`. That case is also the ordinary absent entry, so it would fail every clean run. Only `os.Remove`'s `fs.ErrNotExist` is used.
  - No rows for already removed and unremovable empty directory: neither is reachable deterministically without root.
- **F4**: the table has an `inFirst` field and a row `NewEmptyDirectoryInFirstDirectoryFailsAndIsRemoved`.
- **F5**: row `NewSymlinkFailsAndIsKept`, skipped on Windows in the loop before `t.Run`.
- **F6**: `checkNewGitEntries` loops over `dirs` in order, not over the map.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: the F3 proposal's `Lstat` wording would fail clean runs. Already handled, as described under F3.
- fix-proposal, LOW: the unremovable-directory message said "empty directory" when `ReadDir` failed. Fixed in the follow-up commit.
- fix-proposal, LOW: `Main` itself stays untested, since it needs a `*testing.M`. Informational; no change possible.

## Disputed

None.

## Deferred

None.
