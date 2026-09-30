# Card 1 — leased-remote-delete

**Edit:**
- `internal/gitrepo/push.go`
- `internal/gitrepo/deleteremotebranch_integration_test.go`

**Create:**
- `internal/gitrepo#Repo.DeleteRemoteBranchLeased`

**Uses:**
- `internal/gitrepo#Repo.DeleteRemoteBranch`

**Intent:**
Add a leased variant of the remote branch deletion, so fabric's gate can delete a remote branch only while it still sits at the SHA a pre-flight probe observed.
`DeleteRemoteBranchLeased` validates `expectSHA` with the package's existing SHA validator (returning `ErrInvalidSHA` on a malformed one), then runs `git push --force-with-lease=refs/heads/<branch>:<expectSHA> <remote> --delete <branch>` through `runChecked`, wrapping any failure as `gitrepo: git push --delete (leased at <expectSHA>): %w`.
Unlike `DeleteRemoteBranch`, an already-absent remote ref is not an idempotent success here: the lease names a SHA the caller saw, so a branch that moved or vanished since is a failed lease and returns an error.
Update `push.go`'s file header, which enumerates the push surface, to name the new method.

Extend `deleteremotebranch_integration_test.go`, reusing its bare-remote/clone fixtures:
- a lease at the remote's current tip deletes the branch, and `ls-remote --heads` no longer lists it;
- a stale lease (the remote branch advanced by a second clone after the SHA was read) returns an error, and the remote branch still points at the advanced tip;
- a lease against an absent remote branch returns an error;
- a malformed `expectSHA` returns `ErrInvalidSHA` without touching the remote.

**ImpactSummary:** additive method on `gitrepo.Repo`; `DeleteRemoteBranch` and its callers are unchanged.
