# Two-repository hubs: status commits skip during weft replay, and the layout probe spams and costs git calls

`two-repo-merge-probe-regression` · Task · bug, fabric · draft

## What was seen (2026-11-19, reading f06acf39 against a live two-repository hub)

- `MergeStateOperation` in `internal/fabricengine/recordsduringmerge.go`, reached through `mergestateactive.go:29`, now reports `CHERRY_PICK_HEAD`, `REVERT_HEAD` and `rebase-*` as well as a merge, for a weft worktree too.
  Fabric's own weft replay runs `git cherry-pick` (`internal/gitrepo/reset.go:72`), so a status commit probing during a replay skips its transition with a warning instead of waiting on the write lock.
  A run's last transition skipped this way is never caught up, and a crashed replay's leftover state makes every transition skip.
  `lym loom commit-records`, batten cancel, leftover pairs and tender run intents also skip or refuse in those states.
- `oneRepositoryAt` runs two to four git commands per call and sits inside `RecordsWorktree`, `RecordsWorktreePath`, `RecordsRepoRoot` and `recordsBranchIn`, some in loops.
  For a path with no worktree on disk, such as a project or a task not started, it logs `fabricengine: reading the layout of a worktree failed; reading it as two repositories`, several times on every `lym board get` and `list`.

## What to do

- In a two-repository hub, the merge-state probe behaves as before f06acf39: only a merge counts; a cherry-pick, revert or rebase in weft does not make a status commit skip.
- The layout probe treats a path with no worktree on disk as two-repository without a warning, and its answer is cached per worktree for the process, so repeated calls run no git.
- Add tests for both: a status commit during a weft cherry-pick in a two-repository hub commits; a layout read of a missing path logs nothing.

One-repository behaviour is out of scope: project `single-repo-hub` is redesigning it.
