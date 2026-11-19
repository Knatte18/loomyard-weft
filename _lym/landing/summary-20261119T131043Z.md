# Two-repo hubs: weft cherry-picks commit; layout probe quiet, cached

Two regressions from f06acf39 hit two-repository hubs.

## What changes

- **Merge-state probe.**
  In a two-repository worktree, `MergeStateOperation` (`internal/fabricengine/recordsduringmerge.go`) again reads only a merge:
  a live `MERGE_HEAD` or a conflicted index.
  A cherry-pick, revert or rebase in the weft no longer counts.
  Fabric's own weft replay cherry-picks under the write lock the status commit waits on.
  Reading the replay as a merge made the commit skip its transition instead of waiting for the lock,
  and a crashed replay's leftover `CHERRY_PICK_HEAD` made every transition skip.
  `MergeStateActive`, and through it `lym loom commit-records`, batten cancel, leftover pairs and tender run intents, follow.
  One-repository worktrees are unchanged.
- **Layout probe.**
  `oneRepositoryAt` (`internal/fabricengine/onerepository.go`) answers false for a path with no `.git` entry,
  such as a project or a task not started, without running git and without logging.
  Before, `lym board get` and `list` logged `reading the layout of a worktree failed` several times for each such path.
  For a worktree on disk the answer is kept in a process-wide map, so `RecordsWorktree`, `RecordsWorktreePath`,
  `RecordsRepoRoot` and `recordsBranchIn` run the probe's git commands once per worktree, not on every call.
- **Docs.** The `fabricengine` package doc describes both behaviours.

## Design decisions

- A path with no worktree on disk is not cached, so a worktree created there later in the same process is read then.
- A failed read is logged and not cached, so a transient git failure does not fix a wrong layout for the whole process.
- Fabric clears the whole cache whenever it commits a `.lym-anchor` marker, seeds the one-repository board,
  or retires the board's marker.
  These are the only fabric writes that change what the probe answers.
  An operator who switches a one-repository prime to a branch without the marker mid-process is not seen until the next process.

## Tests

- `TestCommitStatusSeam_Real` (`internal/loomcli`) gains the step `records cherry-pick in progress commits the transition`.
  The real probe runs while a stopped cherry-pick sits in the records sibling.
  The pick then ends before the commit, as a replay does, and the transition commit has to land.
- `TestMergeStateActive` (`internal/fabricengine`) gains the step `weft cherry-pick in progress reports false`.
  It checks that `MergeStateActive` reports false and `MergeStateOperation` names no operation.
- `TestOneRepositoryAt_NoWorktreeOnDiskReadsAsTwoRepositoriesQuietly` (`internal/fabricengine`)
  reads a missing path and a directory without `.git`, and checks that each answers false and logs nothing.
  It swaps the process-global log output, so it does not run in parallel.
