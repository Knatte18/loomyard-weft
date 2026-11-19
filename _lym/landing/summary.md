# Two-repo hubs: no status skip in weft replay; quiet cached layout probe

In a two-repository hub, a status commit that probed during fabric's own weft replay skipped its transition with a warning.
The replay runs `git cherry-pick` (`internal/gitrepo/reset.go:72`) under the write lock the commit waits on.
A run's last transition skipped this way was never caught up, and a crashed replay's leftover state made every transition skip.
The cause was `MergeStateOperation` reporting `CHERRY_PICK_HEAD`, `REVERT_HEAD` and `rebase-*` for a weft worktree as well as a merge.

The layout probe `oneRepositoryAt` also ran two to four git commands per call, inside `RecordsWorktree`, `RecordsWorktreePath`, `RecordsRepoRoot` and `recordsBranchIn`.
For a path with no worktree on disk, such as a project or a task not started, it logged `reading the layout of a worktree failed` several times on every `lym board get` and `list`.

## What changes

- `MergeStateOperation` (`recordsduringmerge.go`) and so `MergeStateActive` count only a merge in a two-repository worktree: a live `MERGE_HEAD` or a conflicted index.
  A weft cherry-pick, revert or rebase no longer makes a status commit skip.
- One-repository behaviour is unchanged: there a cherry-pick, revert or rebase still counts, and a merge counts only when its index holds a records path unlike HEAD's.
  That layout is being redesigned under project `single-repo-hub`.
- `oneRepositoryAt` (`onerepository.go`) answers two repositories without running git or logging when the path has no `.git` entry.
  That answer is not kept, so a worktree created there later is read then.
- Any other answer is kept per worktree for the process, so a repeated call runs no git.
  A failed read still answers two repositories, logs at Warn and is not kept.
- The git reading moved to `readOneRepositoryLayout`, which returns its error to `oneRepositoryAt`, so one Warn replaces the per-step warnings.

## Design decisions

- The cache is bounded by the process.
  A fabric write that adds or retires a `.lym-anchor` marker (`anchormarker.go`, `clone_onerepository.go`) calls `forgetLayouts`, which drops every kept answer.
  A marker another process adds or retires is seen only by a later process.
- A read that overlaps a clear must not leave a stale answer.
  `oneRepositoryAt` reads the `layoutClears` counter before the git read, stores the answer, and deletes the key when the counter changed.
  Checking before the store would leave a window between the check and the `Store`.
- `doc.go`, `MergeStateActive` and `MergeStateOperation` docs state the two-repository and one-repository rules and the process bound.

## Tests

- A status commit during a weft cherry-pick in a two-repository hub commits (`wiring_commitstatus_integration_test.go`, `mergestateactive_integration_test.go`).
- A layout read of a missing path logs nothing (`onerepository_quiet_test.go`).
- `onerepository_cache_test.go` pins the kept answer per worktree.
  It is serial, because every marker commit clears the process-wide map.

## To check by hand

- `layout-cache-untested` (class design, MEDIUM) was open after review, in `internal/fabricengine/onerepository_cache_test.go` and `onerepository.go`.
  A test now pins the kept answer, its drop by a marker commit and the uncached failed read.
  Confirm it covers the clear-overlapping-read path in `oneRepositoryAt`, which the counter check guards.
- No regression check ran and no live two-repository hub was exercised after the change.
