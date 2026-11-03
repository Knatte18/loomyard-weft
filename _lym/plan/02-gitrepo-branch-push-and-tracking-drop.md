# Card 2 — gitrepo-branch-push-and-tracking-drop

**Create:**
- `internal/gitrepo#Repo.PushBranch`
- `internal/gitrepo#Repo.DropRemoteTracking`
- `internal/gitrepo_test#TestPushBranch`
- `internal/gitrepo_test#TestDropRemoteTracking`

**Edit:**
- `internal/gitrepo#Repo.pushPlainWithRetry` -> `func (r *Repo) pushPlainWithRetry(remote, branch string) error`
- `internal/gitrepo#Repo.PushRebaseFree`
- `internal/gitrepo#Repo.pushWithRebaseRetry`

**Extends:**
- `internal/gitrepo/doc.go`

**Uses:**
- `internal/gitrepo#Repo.withRemoteRefs`

**Intent:** Two gitrepo primitives fabric's Remove (D4) and Add (D8) need, each under the repository's remote-refs lock as `PATTERN-remote-refs-lock` requires for a write of a remote-tracking ref.

- `pushPlainWithRetry` gains `remote` and `branch`.
  Both empty is today's push of the checked-out branch (`refuseMarkedCurrentBranch`, then `push` with `push.autoSetupRemote=true`); `PushRebaseFree` and `pushWithRebaseRetry` (twice: the first push and the retry after its rebase), its callers today, pass both empty and are otherwise unchanged.
  A named branch refuses a branch carrying the local-only mark with `ErrLocalBranch` before git runs, then pushes `refs/heads/<branch>:refs/heads/<branch>` to `remote`, the branch need not be checked out; the retry ref is that branch, so each attempt runs under the remote-refs lock and every backoff outside it, as today.
- `PushBranch` calls `pushPlainWithRetry(remote, branch)` and maps a rejection to an error wrapping `ErrPushRejected` and an exhausted retry to `ErrPushRetriesExhausted`, as `PushRebaseFree` does; it never pulls or rebases, and it writes the remote-tracking ref as any plain push does.
  It passes the same gitexec pause gate every push passes, adding no gate branch of its own.
- `DropRemoteTracking` deletes `refs/remotes/<remote>/<branch>` and, only when the branch has `branch.<branch>.remote` or `branch.<branch>.merge` configured, unsets both, in one hold of the remote-refs lock.
  An absent tracking ref and an absent upstream config are no error, so the call is idempotent; any other git failure is returned.

Add both to `internal/gitrepo/doc.go`'s list of operations that take the remote-refs lock, and to the `push.go` file header's list of the push surface, whose "push-shaped remote calls" sentence also names the tracking-ref drop.

Tests, `tier1`, over a hermetic repository with a bare origin (the existing `gitrepo_test` push fixtures):
- `TestPushBranch`: a branch not checked out, ahead of origin, lands at its local tip and its tracking ref moves there; a branch origin has diverged from returns an error matching `ErrPushRejected` and leaves origin's tip; a marked branch returns `ErrLocalBranch` and origin is untouched.
- `TestDropRemoteTracking`: a branch with a tracking ref and upstream config loses both, `git config branch.<b>.merge` then reading unset; a second call on the same branch returns nil.

**ImpactSummary:** Re-signs one private helper whose two callers `PushRebaseFree` and `pushWithRebaseRetry` are on this card and keeps its behaviour; adds two methods no existing caller reads.

**Commit:** `2: gitrepo pushes a named branch and drops its tracking under the remote-refs lock`
