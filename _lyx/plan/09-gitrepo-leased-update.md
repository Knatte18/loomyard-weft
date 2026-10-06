# Card 9 — gitrepo-leased-update

**Create:**
- `plan:internal/gitrepo#Repo.UpdateRemoteBranchLeased` -> `func (r *Repo) UpdateRemoteBranchLeased(remote, branch, sha, expectSHA string) error`
- `plan:internal/gitrepo#Repo.CommitsNotIn` -> `func (r *Repo) CommitsNotIn(tip, base string) ([]string, error)`
- `plan:internal/gitrepo#ErrLeaseRejected` -> `var ErrLeaseRejected error`

**Edit:**
- `internal/gitrepo/push.go`
- `internal/gitrepo/ancestry.go`
- `internal/gitrepo/doc.go`
- `internal/gitrepo/push_test.go`
- `internal/gitrepo/ancestry_integration_test.go`
- `cmd/lyx/gitrepoboundary_test.go`

**Intent:**
Add the two git primitives the guarded remote update of card 11 and the remote-divergence read of card 10 need, on the client side `PATTERN-gitrepo-client-boundary` assigns each.

`UpdateRemoteBranchLeased`, in `push.go`, moves `branch` on `remote` to `sha` through `gitexec` with `git push --force-with-lease=refs/heads/<branch>:<expectSHA> <remote> <sha>:refs/heads/<branch>`, succeeding only while the remote branch still sits at `expectSHA`.
It mirrors `DeleteRemoteBranchLeased`: both SHAs must be valid hex object names or `ErrInvalidSHA` is returned before any git spawn, it runs through `runChecked`, and a failed lease (the branch moved or vanished) is an error, never an idempotent success.
A failed lease returns the new sentinel `ErrLeaseRejected`, bare so callers match it with `errors.Is`, classified from git's `stale info` stderr text the way `PushRebaseFree` classifies `ErrPushRejected` from its triggers (the package doc's locale caveat applies).
Every other failure, a network or authentication error or a hook rejection, returns the wrapped git error and never the sentinel; that split is what lets the caller give a moved remote and an unreachable one their different ways forward.
It is the only force-shaped push in the package, so its doc comment states that fabric's gate is its only caller and that it never takes a lease it did not read itself.

`CommitsNotIn`, in `ancestry.go`, returns the SHAs of the commits reachable from `tip` and not from `base`, newest first, by running `git rev-list base..tip` through `runChecked`.
It is CLI-bound beside `IsAncestor`, which is a `git merge-base --is-ancestor` call: the package doc records ancestry walks as measured and reverted from go-git, so this method does not reopen that choice.
Both SHAs must be valid hex object names or `ErrInvalidSHA` is returned before any git spawn, and an unknown `tip` or `base` is an error naming it.

Update `internal/gitrepo/doc.go`'s method list for both, adding each to its enumeration of the CLI-bound methods that sit on `runChecked`.
`cmd/lyx/gitrepoboundary_test.go` pins the set of `*Repo` methods whose body calls `r.run` or `r.runChecked`, so both method names join its `gitrepoPinnedRunBoundMethods` list in this card, or `go test ./...` fails.

Tests:
in `push_test.go`, over the bare-remote fixture: a leased update with the remote's current tip moves the remote branch (including backwards to an ancestor); a lease naming a stale SHA fails with `errors.Is(err, ErrLeaseRejected)` and leaves the remote branch where it was; a failure that is not a lease loss (a `pre-receive` hook on the bare remote that exits non-zero) returns an error for which `errors.Is(err, ErrLeaseRejected)` is false; an invalid SHA returns `ErrInvalidSHA` with no git spawn.
In `ancestry_integration_test.go`: `CommitsNotIn` lists exactly the commits on a side branch past the merge base, newest first, returns none when `tip` is an ancestor of `base`, and errors on an unknown ref.

**ImpactSummary:** Two new Repo methods and one sentinel, a leased force push and a rev-list read, both through gitexec; no existing method changes, and the boundary test's pinned list gains both names.
