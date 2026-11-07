# Read `worktree list` in-process through `internal/dotgit`

`fabricengine.List` and webster's sibling-worktree read no longer start `git worktree list --porcelain`.
They read the repository's files through a new `dotgit.ListWorktrees`.
`lym` calls `List` three or more times in each `Add` and `Remove`, and the spawn was about 15 percent of `internal/fabricengine`'s git processes.

## The reader

`dotgit.ListWorktrees(dir)` returns `[]dotgit.Worktree` (path, HEAD SHA, full branch ref, locked, prunable) in the order git 2.43 through 2.53 prints:

- It finds the repository by walking up from any directory inside a worktree through `FindRoot`. A `dir` that does not exist is an error with no walk-up, so the `List` fallbacks in `junction.go` and `scratchlink.go` still fire on a gone pair path.
- The main worktree comes first, its path being the common dir resolved through symlinks, less `.git`.
- Linked worktrees come from `worktrees/<id>/`: `gitdir`, `HEAD` and `locked`. They are sorted by path, compared case-folded when `core.ignorecase` is true.
- A relative `gitdir` (git 2.48+) resolves against its admin entry. An absolute one is taken as written.
- An entry whose `gitdir` is missing, unreadable or empty is left out. An entry with no `HEAD` is detached at the all-zero SHA.
- An unlocked entry whose `.git` is gone is prunable. A locked one never is, and file age plays no part.
- HEAD resolves through loose refs, then `packed-refs`, in the common dir. A branch with no ref yet gives the all-zero SHA, 64 digits for a sha256 repository.
- A bare repository stays an error.
- A repository whose `extensions.refStorage` is not `files` returns the new `dotgit.ErrRefStorage`.

The package stays standard-library only and spawns nothing.
It reads only the `core` and `extensions` settings it needs from the repository's `config`, and follows no include.

## Callers

- `fabricengine.List` maps the entries onto `WorktreeEntry` unchanged: branch short name or `(detached)`, `Main` on the first entry. On `ErrRefStorage` it falls back to its existing porcelain parse.
- `websterengine`'s `otherWorktrees` reads the roots through `dotgit`. On `ErrRefStorage` it falls back to its existing scan of the `worktree` lines.

## Tests

- `internal/dotgit/worktrees_test.go` (tier 0) is one table over hand-laid repositories. It covers each rule above, the refusals and a symlinked start.
- `internal/dotgit/worktrees_git_test.go` (tier 1) compares the reader with git's own porcelain on a repository git wrote, from the main worktree, through a symlink and from a linked worktree's subdirectory. That repository holds packed-only and loose branches, detached, locked with and without a reason, prunable, gitdir-removed and HEAD-removed entries. A second test covers a relative `gitdir` and skips below git 2.48, naming the version.
- `fabricengine`'s `TestList_RefStorage` and `websterengine`'s `TestOtherWorktrees` read a files-backed repository with `PATH` emptied, which proves no git process starts. They read a reftable repository through the fallback, skipping below git 2.45. `TestList_RefStorage` also checks that `List` from a subdirectory equals `List` from the root.
- `TestList_NotAGitRepo` became `TestList_Errors`, which adds a missing directory inside a repository. The old not-a-repository row now matches `dotgit`'s error text, not git's stderr.
- `dotgit`'s `TestMain` moved to the external test package and calls `gitkit.HermeticGitEnv`, because `gitkit` reaches `dotgit` through `lymcwd`.
- `gitkit.SkipBelowGitVersion` is the shared version skip.

## Design decisions

- Bare detection checks whether the start directory is itself a git dir before walking up, so `List` on a bare repository's directory still says "bare repositories are not supported".
- A symbolic-ref chain follows up to five links, as git does, and the branch reported is the last ref of the chain, as git's resolver names it.
- `PATTERN-leaf-packages`' `internal/dotgit` background now lists the worktree entries, refs and config keys `dotgit` parses. The `PATTERN.md` index line keeps its wording, as the record's `## PATTERN changes` gives it.
