# Read the worktree list in-process through internal/dotgit

`fabricengine.List` and the worktree read in `websterengine` no longer start `git worktree list --porcelain`.
That call is about 15 percent of `internal/fabricengine`'s git processes and about 5 s of the package's wall,
and `lym` calls `List` three or more times in each `Add` and `Remove`.

## What changes

- `internal/dotgit` gains `ListWorktrees`, with the `Worktree` entry type and the `ErrRefStorage` error.
  It takes any directory inside a worktree, finds the repository by walking up as git does, and returns the main worktree first,
  then each `worktrees/<id>/` entry.
  Per entry it reads `gitdir`, `HEAD` and `locked`, and resolves `HEAD` through the loose ref, then `packed-refs`.
  It stays standard-library only and spawns nothing.
- `fabricengine.List` reads through `ListWorktrees`; its `WorktreeEntry` result and callers are unchanged.
- `websterengine`'s `otherWorktrees` reads through `ListWorktrees` via a new `worktreeRoots`.
- `PATTERN-leaf-packages` lists the worktree entries, refs and config keys `dotgit` now parses.
- `gitkit` gains `SkipBelowGitVersion`, which the relative-`gitdir` parity test uses.

## Design decisions

The reader matches `git worktree list --porcelain` as git 2.53.0 prints it, and behaves the same at the git 2.43.0 floor except for relative `gitdir`:

- The main worktree's path is resolved through symlinks; a linked path is the directory of the `.git` its `gitdir` names.
- Linked worktrees follow the main one sorted by path, case-folded where `core.ignorecase` is true.
- A relative `gitdir` (git 2.48 and later) resolves against its `worktrees/<id>/` directory.
- An entry whose `gitdir` is missing or unreadable is left out.
- An entry is prunable when it is not locked and its `gitdir` names a `.git` that does not exist; age plays no part.
- An entry with no `HEAD` file is listed detached with the all-zero SHA, and `List` never fails on it.
- A detached `HEAD` gives `List` the branch value `(detached)`; a branch whose ref does not exist yet gives the all-zero SHA.
- A bare main repository stays an error.
- A directory that does not exist is an error with no walk-up,
  so the fallbacks in `fabricengine/junction.go` and `fabricengine/scratchlink.go` still fire on a pair path that is gone.

A repository whose `extensions.refStorage` is not `files` has no ref files to read.
`ListWorktrees` refuses it with `ErrRefStorage`, and `List` and `worktreeRoots` fall back to `git worktree list --porcelain`.
The porcelain path stays in both callers for that case alone.

## To check by hand

No regression check or integration suite ran for this task.
Run the `internal/dotgit`, `internal/fabricengine`, `internal/websterengine` and `internal/gitkit` tests.
The relative-`gitdir` parity test skips below git 2.48, so run it once under 2.48 or later.
