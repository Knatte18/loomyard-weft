# Card 9 — gitrepo-leased-update

**Create:**
- `plan:internal/gitrepo#Repo.UpdateRemoteBranchLeased` -> `func (r *Repo) UpdateRemoteBranchLeased(remote, branch, sha, expectSHA string) error`
- `plan:internal/gitrepo#Repo.CommitsNotIn` -> `func (r *Repo) CommitsNotIn(tip, base string) ([]string, error)`

**Edit:**
- `internal/gitrepo/push.go`
- `internal/gitrepo/ancestry.go`
- `internal/gitrepo/doc.go`
- `internal/gitrepo/push_test.go`
- `internal/gitrepo/ancestry_integration_test.go`

**Intent:**
Add the two git primitives card 10's guarded remote update needs, on the client side `PATTERN-gitrepo-client-boundary` assigns each.

`UpdateRemoteBranchLeased`, in `push.go`, moves `branch` on `remote` to `sha` through `gitexec` with `git push --force-with-lease=refs/heads/<branch>:<expectSHA> <remote> <sha>:refs/heads/<branch>`, succeeding only while the remote branch still sits at `expectSHA`.
It mirrors `DeleteRemoteBranchLeased`: both SHAs must be valid hex object names or `ErrInvalidSHA` is returned before any git spawn, it runs through `runChecked`, and a failed lease (the branch moved or vanished) is an error, never an idempotent success.
It is the only force-shaped push in the package, so its doc comment states that fabric's gate is its only caller and that it never takes a lease it did not read itself.

`CommitsNotIn`, in `ancestry.go`, returns the SHAs of the commits reachable from `tip` and not from `base` (what `git rev-list base..tip` lists), newest first, as a go-git local read like `IsAncestor`, with the same retrying object lookup.
An unknown `tip` or `base` is an error naming it.

Update `internal/gitrepo/doc.go`'s method list for both.

Tests:
in `push_test.go`, over the bare-remote fixture: a leased update with the remote's current tip moves the remote branch (including backwards to an ancestor); a lease naming a stale SHA fails and leaves the remote branch where it was; an invalid SHA returns `ErrInvalidSHA` with no git spawn.
In `ancestry_integration_test.go`: `CommitsNotIn` lists exactly the commits on a side branch past the merge base, newest first, returns none when `tip` is an ancestor of `base`, and errors on an unknown ref.

**ImpactSummary:** Two new Repo methods, a leased force push through gitexec and a go-git ancestry read; no existing method changes.
