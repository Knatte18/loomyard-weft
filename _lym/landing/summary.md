# Call a rejected push a divergence only when a fetch shows one

A push the remote rejected was always recorded as `not_fast_forward` and mailed with the `git reset --keep origin/main` or `git merge origin/main` way forward, whether or not origin held anything the branch lacked.
Two pushes of the board from one hub overlapped, the loser ran out of retries, and creel mail told the orch about a divergence that did not exist.
Following that advice would have dropped a local commit or merged a history that never existed.

A rejection that outlasts the retry backoff is now judged by a fresh fetch, in `failRejection` in `internal/fabricengine/pushclassified.go`.

- Origin holds commits the branch lacks: `not_fast_forward`, with the local commits and the divergence way forward, as before.
- Origin holds nothing the branch lacks: class `other`, whose way forward is a plain retry of the side's push step.
- The fetch fails, or origin's tip cannot be read after it: class `other` with that error joined to the rejection.
  Origin's tip is the branch's upstream, or origin's branch of the same name for a branch never given an upstream, so a branch with neither is not called a divergence.

This applies on the code, records and bolt sides.
A replay conflict or a destructive refusal stays `not_fast_forward` unjudged, since a replay runs only when origin held commits the branch lacked.
The judgment adds one fetch on the failure path after the backoff ran out; it never pushes, merges, rebases or forces, so it changes only a record's class and way forward.

A non-divergent rejection, such as a declining remote hook, still writes a record and sends its one notice, and its error still satisfies `IsPushRejected` and `errors.Is(err, gitrepo.ErrPushRejected)`.

In `internal/gitrepo/push.go`, `PushRebaseFree` now returns `ErrPushRejected` wrapped with git's last rejection line, such as `! [remote rejected] main -> main (pre-receive hook declined)`, so the notice says why git refused.
The sentinel's text drops `(remote diverged)` and reads `gitrepo: push rejected`.
No caller compared the sentinel by identity, so wrapping it changes no match.
The package docs of `fabricengine` and `gitrepo` describe the judged class.

## Decision: no lock around the board's commit and push

The board's pushers are `lym board sync`, which commits under `board.lock` and pushes under `board.push.lock`, and the callers of `Bolt.PushRecorded` and `Bolt.Push` outside it (owner set, hub config commit and reconcile, orch rules, fabric hub-config paths, the anchor marker), which push under neither; the Bolt retry alone takes `.gitrepo-push.lock`.
The board checkout is the records repository's main worktree, so every weft push and fetch of the hub works in the same repository and its remote-tracking refs.
The logs and creel journal of the incident are pruned, so the two overlapping processes are not identified by pid; any lock-free pusher, or a weft push's fetch, overlapping a board sync fits the reflog.

No lock is added.
Holding the board write lock across a network push would make every board write wait on origin, and making every Bolt push take `board.push.lock` would deadlock `Bolt.Sync`, which already holds it around its push.
Two overlapping fast-forward pushes of one linear history leave nothing to settle: the existing retry lands the loser, and the judgment keeps a loser that runs out of retries from naming a divergence.
Overlapping pushes still cost a retry, and in the worst case an `other` notice with a plain-retry way forward.

## To check by hand

- Run the integration tests of `internal/fabricengine` and `internal/gitrepo`, including `TestPushFailureClasses`; this description was written without running them.
- A branch with no upstream and no origin branch of its name must record `other`, not `not_fast_forward` (`failRejection`, `internal/fabricengine/pushclassified.go`); check that `TestPushFailureClasses` has a row for it.
- A judgment fetch that fails must record `other` with the rejection line and the fetch error; check that `TestPushFailureClasses` has a row for it.
- Rejection text no longer claims a divergence: check that `internal/gitrepo/push_test.go` asserts the error holds git's rejection line and not `remote diverged`, since the assertion in `internal/loomcli/commitrecords_test.go` matches the old text too.
- The `other` notice must name a plain retry: check that the rejection table in `internal/fabricengine/pushclasses_integration_test.go` asserts the `FailedPushWayForward` text for class `other`.
- `internal/fabricengine/pushanchored_integration_test.go` must not say the rejection surfaces unwrapped, in its header or its diverged-weft test name and comment.
