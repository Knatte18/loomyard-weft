# Card 1 — HeadContains walks a committer-time-bounded window

**Edit:**
- `internal/gitrepo#Repo.HeadContains`
- `internal/gitrepo/blobread_integration_test.go`

**Intent:** Decision D5: when a stencil seed pass finds a source-ahead stencil, `HeadContains` walks HEAD's whole history for a target that is absent from it.
Bound the walk instead.
In its existing shared-lock walk, which card 2 moves into `readGoGit` unchanged, `HeadContains` walks HEAD's history with `git.LogOrderCommitterTime` (newest committer time first) and stops with `false` once the next commit's committer time is more than 24 hours older than the target commit's committer time.
The 24-hour slack is a named unexported constant beside the method.
Its contract is unchanged: true for HEAD and its ancestors, false with no error for an absent sha or an unborn HEAD, `ErrInvalidSHA` for a malformed sha.
Its doc comment states the bound and what it lets through: a descendant whose committer time is more than 24 hours older than the target's, from clock skew on the committing machine, makes it answer false for a commit that is in HEAD; its one caller, `fabricengine.StencilSource`, then logs the "behind the build" drift line instead of "source ahead", and neither blocks.

`blobread_integration_test.go`'s `TestHistoryReads` keeps its existing `HeadContains` contract rows and gains a step for the bound.
The step builds its own repository with `newRepo` and leaves the shared fixture untouched, since the other steps read the shared repository's HEAD and whole history.
Its own fixture commits at least three old commits with `GIT_COMMITTER_DATE` set days before a side-branch target commit and a HEAD commit after it, then deletes the oldest old commit's loose object file.
go-git's committer-time walker loads every parent of a commit before returning that commit, so the bounded walk still reads the newest old commit (HEAD's parent) and that commit's parent; deleting either of those two would fail the bounded walk too, which is why the deleted commit sits at least three generations below HEAD.
`HeadContains(target)` answers `(false, nil)`, which an unbounded walk cannot, since reading the deleted commit would fail.

**ImpactSummary:** `HeadContains` reads fewer commits for an absent target and can answer false for a skewed descendant; `fabricengine.StencilSource` is its one caller and only logs on either answer.

**Commit:** 1: gitrepo: HeadContains stops at a committer-time bound
