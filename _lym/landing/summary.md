# gitrepo: run a push's transfer outside the remote-refs lock

A slow or retried push no longer blocks every other fetch, pull and board claim on the weft repository.

The Bolt checkout is a worktree of the weft repository, so Bolt and every pair's weft side share one remote-refs lock.
Every `gitrepo` push held it for each whole attempt, `git send-pack` included.
On 2026-11-04 a detached weft push of one pair held it for 60–80 s per attempt while GitHub answered `Internal Server Error`, and every Bolt claim, Bolt write and weft fetch queued behind it on each create.

## What changes

Every `gitrepo` push now sends to the remote's push URL.
A push to a URL writes no remote-tracking ref, so the transfer, retries included, runs outside the lock.
Once the transfer succeeded, the push takes the lock for a short step that:

- moves the branch's tracking ref to the pushed commit, or deletes it after a branch delete;
- sets the upstream (`branch.<name>.remote` and `.merge`) for a first push of the checked-out branch, as `push.autoSetupRemote` did.

The new `pushdestination.go` resolves where a push lands and records it.
It replaces the `-c push.autoSetupRemote=true` flag, so a push now names its branch explicitly.
The checked-out branch goes to its push remote, `remote.pushDefault`, its upstream remote or `origin`, in git's order.
`push.go`, the delete and leased-update pushes and the archive's branch push all take the new path.

## Design decisions

- **Judges of origin are unchanged.**
  `PATTERN-remote-refs-lock` says a judge of origin holds the lock from reading origin to its verdict.
  A judge still blocks every tracking-ref write between its read and its verdict.
  A push transfer may now land during that window, and is judged against the origin read before it.
  The pattern file and the `gitrepo` and `fabricengine` docs say so.
- **A recording step that fails is logged at Warn and never fails the push.**
  The transfer has already landed, so reporting failure would make a caller retry or roll back a push that succeeded.
  A tracking ref left stale is corrected by the next fetch, except after a delete, where nothing prunes it.
- **A tracking ref that a later fetch moved past the pushed commit stays.**
  The record step keeps a descendant of the pushed sha.
- **No tracking ref is written when none can be named.**
  That covers a remote given as a path or URL, and a remote whose fetch refspecs map only other branches.
- `PushReflogSubject` is now one exported constant in `gitrepo`, which `fabricengine/reflog.go` reads to find the last push boundary.

## To check by hand

Not run against GitHub; the tests use local bare remotes.

- Push to a real remote while another process fetches or claims on the same weft repository, and confirm the fetch does not wait for the transfer.
- Review `recordLandedPush` in `push.go`: a failed record after a landed delete leaves a stale tracking ref until something drops it.
- The default push's remote choice is tested against `pushRemote`, `remote.pushDefault`, the upstream remote and `origin`.
  Confirm the precedence matches git for a repository with several remotes.

## Not in scope

The 69 s Open-Window step in the same log is unexplained; its trace was not kept at Info.
Read it at Debug the next time it appears.
