# Push transfers outside the remote-refs lock

`gitrepo`'s pushes held the repository's remote-refs lock for each whole attempt, `git send-pack` included.
The Bolt checkout and every pair's weft side share one lock in the weft repository's common git dir.
A push that hung or retried against a failing GitHub therefore queued every board claim, Bolt write and weft fetch or push behind it, for minutes at a time.

## What changes

Every push method in `internal/gitrepo` now sends to the remote's push URL, read with `git remote get-url --push`, instead of the remote's name.
A push to a URL writes no remote-tracking ref, so the transfer and its retries run without the lock.
Once the transfer succeeds, the push takes the lock for a short local step in the new `pushdestination.go`.
That step moves the branch's tracking ref to the pushed commit, or deletes it after a remote delete.
For a first push of the checked-out branch, it also sets the branch's upstream.

- The tracking ref is the one the remote's fetch refspecs map the branch to, as git's own push picks it.
  A remote argument that names no configured remote is pushed to as a URL or path and records nothing.
- The default push (`Push`, `PushRebaseFree`, `PushCoalesced`) now names its branch explicitly.
  It pushes the checked-out branch to the same-named branch of its push remote, push default, upstream remote or `origin`, in that order.
  It sets the upstream itself where `push.autoSetupRemote` used to.
- A plain push leaves a tracking ref alone when a fetch has already moved it to a descendant of the pushed commit.
  The leased update and the deletes write their result unconditionally.
- Fetch, pull, the rebase-retry's `pull --rebase` and `DropRemoteTracking` still run wholly under the lock.

## Why the origin judge stays correct

`ReconcileOrigin` holds the lock from its `ls-remote` pair to its last branch verdict.
The write it must exclude is the tracking-ref write, not the transfer.
If a push's tracking ref moved between origin's read and the verdict, the reflog would hold a commit origin seemed to lack, and the verdict would read rewound.
That write still waits for the lock.
A transfer that lands while the verdicts run is judged against the origin read before it, which reads as ahead, never a rewind.
A transfer that lands before the read shows as a moved head: the reconcile fetches it, and the push's later write finds the ref already there.

## Design decisions

- The tracking-ref write is a plain `update-ref`, not a compare-and-swap against the value read before the push.
  Under the lock no other lym writer can race it, and a failed swap after a successful push would report a landed push as failed.
- The commit recorded is the one the source names when the push starts.
  A commit added to the branch during the transfer leaves the tracking ref one commit behind origin, and the next fetch corrects it.
- The board entry's Open-Window observation needs a Debug trace from a future run, so this change does nothing about it.

## Tests

- `remoterefslock_integration_test.go` replaces the push rows of the old waits-on-the-lock test with a table over `PushRebaseFree`, `PushBranch`, `UpdateRemoteBranchLeased`, `DeleteRemoteBranchLeased` and `DeleteRemoteBranch`.
  Each row holds the lock, sees the transfer land on the bare remote, checks that the tracking ref and upstream have not changed, then releases and checks the recorded result.
- A second test fetches past the pushed commit while the push waits for the lock, and checks that the later tracking ref survives.
- The `PATTERN-gitrepo-client-boundary` pinned method list in `cmd/lym/gitrepoboundary_test.go` gains the three new CLI-bound helpers.

`PATTERN-remote-refs-lock` and its background file, the `gitrepo` and `fabricengine` package docs, and the reconcile and archive doc comments now describe the split.
