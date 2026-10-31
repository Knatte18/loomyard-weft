# A rejected push is a divergence only when a fetch shows origin ahead

A push whose rejection outlasted the retry was always recorded as `not_fast_forward`.
Its notice then said "origin/<branch> holds commits this branch lacks" and advised `git reset --keep` or `git merge`, even when origin held nothing the branch lacked.
That is what happened to the board's `main` on 2026-10-30: two pushes from this hub overlapped, the loser's retry failed, and the mail named a divergence that did not exist.
Following its advice would have dropped a local commit or merged a history that was never there.

## What changes

- `pushClassified` (`internal/fabricengine/pushclassified.go`) judges a rejection the retry did not settle with a fresh fetch, in the new `failRejection`, on the code, records and bolt sides alike.
  - Origin holds commits the branch lacks: `not_fast_forward`, with the local commits and the merge or reset way forward, as before.
  - Origin holds nothing the branch lacks: `other`, whose way forward is a plain retry of the side's push step. The error says a fetch found origin holding nothing this branch lacks.
  - The fetch fails: `other`, with the fetch error joined to the rejection, since origin's state is unknown.
  - A replay conflict or destructive refusal stays `not_fast_forward` unjudged, because a replay only runs when origin held commits the branch lacked.
- `gitrepo.PushRebaseFree` (`internal/gitrepo/push.go`) now wraps `ErrPushRejected` with git's rejection line, such as `! [remote rejected] main -> main (pre-receive hook declined)`. Before, it returned the bare sentinel.
  The sentinel's text drops `(remote diverged)` and is now `gitrepo: push rejected`, because a rejection alone says nothing about what origin holds.
  `errors.Is` matching is unchanged, and no caller compared the sentinel by identity.
- The `fabricengine` package doc's push-classification section, and the doc comments of `pushClassified`, `PushAnchored` and `PushRebaseFree`, describe the judged class.

A non-divergent rejection that never lands, such as one from a declining hook, still writes a record and sends its one notice, under `other`.
Its error still satisfies `IsPushRejected`.

## Which processes overlap, and why no new lock

The board checkout is the records repository's main worktree.
`lym board sync` commits under `board.lock` and pushes under `board.push.lock`, so two board syncs never overlap each other.
These other callers push the board with neither lock:
- the owner setter
- the hub config commit and reconcile
- `lym orch rules`
- fabric's hub-config paths
- the anchor marker

Separately, every weft push and fetch of the hub runs in the same repository and touches the same remote-tracking refs.
Only the Bolt retry takes `.gitrepo-push.lock`.
The hub's logs and creel journal for 2026-10-30 have been pruned, so the two processes cannot be named by pid.
Any of these lock-free pushers, or a weft push's fetch, overlapping a board sync fits origin's reflog.

The fix adds no lock that covers both commit and push:
- Holding `board.lock` through the push would make every board write wait on the network.
- Making every Bolt push take `board.push.lock` would deadlock `Bolt.Sync`, which already holds that lock around its own push.

Two overlapping fast-forward pushes of one linear history leave nothing to settle.
The existing retry lands the push that lost the race, and the fetch judgment keeps one that runs out of retries from claiming a divergence.

## Design decisions

- A failed judgment fetch is always recorded as `other`, never `transport` as decision D1 first said: `gitrepo.Repo.Fetch` reports a failed fetch by its exit code rather than as a `*gitexec.GitError`, so an unreachable origin cannot be told apart there.
- The judgment adds one fetch, only on the failure path after the backoff has run out. It never pushes, merges, rebases or forces, so it can change only a record's class and way forward.

## Tests

- `TestPushFailureClasses` covers the new judgment in one table, `ARejectionThatOutlastsTheBackoffWithOriginHoldingNothingTheBranchLacksIsNoDivergence`, with code, records and bolt rows.
  It replaces the two subtests that pinned a declining hook as `not_fast_forward`.
  Each row checks for an `other` record quoting git's refusal line, exactly one notice that names no divergence and no merge, and the full backoff waited out before the record.
- The declining-hook assertions in `pushanchored_integration_test.go` and `boltpullfirst_integration_test.go` now expect `other`.
- The real-divergence cases are unchanged and still record `not_fast_forward`: a code push that another machine got to first, and records or bolt replay conflicts.
- The `loomcli` commit-records row matches the new sentinel text.
