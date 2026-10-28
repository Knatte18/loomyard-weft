# fabricengine: retry a rejected push before reporting a divergence

A push that loses a race with another push of the same branch now lands on a later attempt instead of writing a `not_fast_forward` record and mailing a notice.
The seen case was two asynchronous pushes of one branch on one machine: the loser's rejection, or a fetch it made while the winner held the tracking ref, named a divergence the winner's landing had already removed.
The records notice then claimed "another machine wrote the same paths" with `paths: none`.

Closes #879

## What changes

- `pushClassified` (`internal/fabricengine/pushclassified.go`) runs a bounded retry loop, `settleRejection`, before it records a rejection, for the code, records and bolt sides.
  - Code side: each attempt (`repushOverOriginContained`) fetches and pushes again only while origin's tip is contained in the local tip.
    When origin holds commits the branch lacks, it stops at once and records the rejection, with no backoff sleep.
  - Records and bolt sides: each attempt is the side's existing replay retry (`retryWeftPush`, `retryBoltPush`).
  - The loop runs again after a bare rejection or an error no class names, such as a failed fetch.
    It stops on a landing, a deferral, a replay conflict, an anchor stop, an auth refusal or a transport failure, which classify as before.
  - Attempts are spaced by `gitrepo.PushBackoffDelays()`, slept through `SyncOptions.BackoffSleep`.
    A rejection that persists, such as a declining hook, is still recorded and mailed once the schedule runs out.
- `pushRebaseFreeRetryingOnce` is removed; the code side pushes with a plain `PushRebaseFree`, and the new loop covers its one-shot stale-tracking-ref retry.
- `recordsConflictWayForward` (`failedpush.go`) no longer says another machine wrote the paths, since no commit lym writes carries a machine name.
  With conflicting paths it names them and says origin's commits change the same paths; with none it omits the `paths:` clause and says to settle any conflict.
- The `doc.go` push classification section and the affected doc comments describe the retry and the new wording.

## Design notes

- The loop never forces, rebases or merges, and a code-side push runs only while origin's tip is contained in the local tip, so it can only fast-forward origin.
- A rejection that persists keeps the `not_fast_forward` class even when origin is contained in the local tip; reclassifying it was not asked for.
- The race cause is not pinned to one code path.
  The fix is the re-check against a fresh fetch, applied to every side, rather than a lock change.
- Machine names on commits are out of scope.

## To check by hand

- A synchronous push whose rejection persists now waits out the backoff schedule before it returns, the same wait a gone branch or an unreachable origin already takes.
- The race itself was not reproduced against a real concurrent pair of pushes; the tests inject the backoff sleep and simulate the rejections.
