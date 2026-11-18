# Push a records side again instead of reading it as ended on origin

`PushPairAnchored` treats its records side as landed when the push returns nil and the worktree then holds nothing unpushed.
When commits were still unpushed and no fresh deferral record existed, `recordsSideNotLanded` fell through to one answer:
the branch's entry is ended on origin.
That fallback never asked origin.
Two transient states reach it on the prime's `main-weft`, which other runs also commit to:

- a commit made to the records worktree after the push read HEAD and before the `HasUnpushed` check;
- a push that landed but left the tracking ref unmoved, because the tracking-ref write failed and was only logged.

Either one failed a new run at Seed-IDE with "the entry of branch main-weft is ended on origin", and a rerun passed.

## Change

- `recordsSideNotLanded` now looks the branch up on origin after ruling out a deferral.
  A branch origin still holds returns an error wrapping the new `errRecordsUnpushedPastPush`.
  Only a branch gone from origin keeps the "ended on origin" message,
  since the classified push skips a gone branch only when its entry ended.
  A failed lookup returns that lookup's error, never the ended message.
- `PushPairAnchored`'s records side pushes again on `errRecordsUnpushedPastPush`,
  sleeping the push backoff delays through `SyncOptions.BackoffSleep`.
  It returns the error only once the backoff is spent.
  Each retry runs the full classified push, so a deferral, rejection or ended entry found on a retry is reported as before.
- The `PushPairAnchored` doc comment and the package doc in `internal/fabricengine/doc.go` state the retry,
  and that only a branch gone from origin reads as ended.

## Design decisions

- The fix reads origin rather than trusting the local tracking ref, so the ended claim rests on the same evidence the push's own origin check uses.
  The cost is one `ls-remote`, paid only on the not-landed path.
- The retry reuses the push backoff schedule rather than a new constant, matching the other retries in the push path.
- Commits that arrive during the push could be left to their committer's own push.
  The retry pushes them anyway: the caller cannot tell its own commits from another writer's, and pushing records commits early is harmless.

## Tests

`TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` gains `PrimeRecordsCommitDuringThePushIsPushedAgain`.
A one-shot `post-receive` hook on the records bare commits to the prime's records worktree while the push runs.
The subtest asserts that the push returns nil, that origin holds the commit the hook made, and that no failed-push record was written.
The existing ended-entry subtests in `pushclasses_integration_test.go` still expect "ended on origin" for a branch deleted from origin, and the new lookup keeps that.
