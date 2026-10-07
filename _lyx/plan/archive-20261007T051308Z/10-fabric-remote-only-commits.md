# Card 10 — fabric-remote-only-commits

**Create:**
- `internal/fabricengine/remoteonly.go`
- `plan:internal/fabricengine#Fabric.RemoteOnlyCommits` -> `func (f *Fabric) RemoteOnlyCommits() (tip string, commits []string, err error)`
- `internal/fabricengine/remoteonly_integration_test.go`

**Edit:**
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/gitrepo#Repo.CommitsNotIn`

**Intent:**
Add the one read of the remote task branch's divergence that both D7's reset and D8's Publish halt rely on, so the two report the same answer.

`RemoteOnlyCommits`, in the new `remoteonly.go`, fetches the remote and returns the remote task branch's tip and the commits on it that the local HEAD lacks (`CommitsNotIn(tip, <the resolved HEAD SHA>)`, since `CommitsNotIn` takes hex object names only), newest first.
The tip is empty, and the commit list empty, when the remote has no such branch or no remote is configured.
A fetch that fails returns the error; the callers decide whether it refuses or degrades.
It mutates no ref other than the remote-tracking refs the fetch refreshes, and all git runs through `gitrepo`/`gitexec` (`PATTERN-fabric-git`).

Docs: `doc.go` names `RemoteOnlyCommits` and what its empty tip means.

Tests (hubforge hub with its origin, `PATTERN-hubforge-fixtures`):
`remoteonly_integration_test.go`, one table-driven test over the three answers: a remote task branch holding a commit the local HEAD lacks returns that tip and that commit; a remote task branch at or behind the local HEAD returns its tip and no commits; a branch absent on the remote, and a repository with no remote, each return an empty tip and no commits.

**ImpactSummary:** One new exported Fabric method with two consumers in other packages, read-only apart from the fetch's remote-tracking refs; no existing symbol changes.
