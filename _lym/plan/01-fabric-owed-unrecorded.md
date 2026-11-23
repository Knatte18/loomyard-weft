# Card 1 — fabric-owed-unrecorded

**Edit:**
- `internal/fabricengine#PairPush`
- `internal/fabricengine#SyncOptions`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#teardownLockOwedError`
- `internal/fabricengine#recordOwedRecordsPush`
- `internal/fabricengine#recordsSideNotLanded`
- `internal/fabricengine#pushJob.fail`
- `internal/fabricengine#pushJob.deferral`
- `internal/fabricengine#pushJob.checkOrigin`
- `internal/pairteardown#RunPush`
- `internal/pairteardown#PushRun`
- `internal/fabricengine#StatusPushLockWait`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/fabricengine_test#TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`
- `internal/gitrepo#Repo.UpstreamSHA`
- `internal/gitrepo/doc.go`

**Extends:**
- `internal/fabricengine/doc.go`

**Create:**
- `plan:internal/fabricengine#ErrEntryEnded` -> `var ErrEntryEnded error`
- `plan:internal/gitrepo#Repo.DeleteRemoteTrackingRef` -> `func (r *Repo) DeleteRemoteTrackingRef(remote, branch string, ns RefNamespace) error`

**Uses:**
- `internal/fabricengine_test#newDivergedPair`
- `internal/fabricengine#FailedPushesOwedBy`

**Intent:**
Give the pair push the mode D5 names, in which an owed push returns its error and writes no failed-push record.
The pusher (card 5) and Worktree-Teardown (card 10) push in this mode and retry an owed push themselves.

- `PairPush` and `SyncOptions` gain the bool field `OwedUnrecorded`.
  `PushPairAnchored` copies it from `PairPush` into the options every side's push reads.
  The zero value keeps today's behaviour for every other caller.
- In the mode, a failure of an owed class (transport, deferred, teardown_lock) builds its `*PushFailedError` as today
  but writes no record and posts no notice:
  `teardownLockOwedError`, `recordOwedRecordsPush` and `pushJob.fail` skip `recordPushFailure` for those classes.
  `pushJob.fail` keeps writing every other class, so a refused push still leaves its record and notice.
- In the mode, `pushJob.deferral` returns the deferred class's unrecorded `*PushFailedError` instead of nil.
  So `pushClassified` returns the deferral as an error, and `PushPairAnchored`'s records loop returns it as the owed failure:
  the deferral is told from the returned error, never read back from a record, and `recordsSideNotLanded`'s record read
  is reached only outside the mode.
  Its transport branch goes through `recordOwedRecordsPush`, which in the mode writes nothing.
- `recordsSideNotLanded`'s error for a records branch whose entry origin ended wraps the new sentinel `ErrEntryEnded`,
  in every mode, so Worktree-Teardown tells "left to its teardown" from a failed push (card 10).
  Its text is unchanged.
- `pushJob.checkOrigin`, on either side, deletes the local tracking ref of a branch gone from origin whose entry origin ended,
  in every mode, before it skips the branch:
  so card 2's local reads see the absent tracking ref they read as level and held, never a stale one that lacks the local tip.
  It deletes the ref through the new `gitrepo.Repo.DeleteRemoteTrackingRef`.
- `DeleteRemoteTrackingRef` removes only the remote-tracking ref that the remote's fetch refspecs map the branch's origin ref under `ns` to,
  resolved as a push destination resolves its tracking ref, in one hold of the remote-refs lock.
  It touches no config, so the branch keeps its upstream setting and reads as upstream-set with a pruned ref.
  An absent ref, or a remote whose refspecs map no ref, is nil.
  `DropRemoteTracking` is not used, because it also unsets `branch.<branch>.remote` and `.merge`:
  a branch with no upstream reads as ahead and not held in card 2, and its next push passes `checkOrigin`'s no-upstream branch
  and recreates the branch origin deleted.
- `UpstreamSHA`'s error for a set upstream whose tracking ref is absent also wraps `gitrepo.ErrRefNotFound`; its text is unchanged.
  `checkOrigin` reads that error as an upstream-set branch with a pruned ref and goes on to look the branch up on origin,
  where today it returns the error and the push fails.
  So the next push of a branch whose ref it deleted finds it gone with its entry ended and skips it with nil,
  and a pruned ref of a branch origin still holds is pushed as one with its ref present.
- `gitrepo/doc.go`'s remote-refs lock line names `DeleteRemoteTrackingRef` among the methods that run under the lock.
- `pairteardown.RunPush` gains `OwedUnrecorded`, and `PushRun` passes it into `PairPush`.
- `StatusPushLockWait`'s doc names it the wait a worktree's pusher and Worktree-Teardown pass for the push lock and the pair's teardown lock,
  since no transition and no `commit-records` pushes after cards 6 and 7.

So `FailedPushesOwedBy` returns no record of a push made in the mode,
and the owed-push read in `internal/pairteardown/runread.go` leaves `PushOwed` unset (D8):
the act rule's readers keep their behaviour and never see the pusher's retries.
`IsPushOwed` still reads every returned owed error as owed, since it reads the error's class, not the record.
The mode skips only the record of a failure that is retried anyway; it lets no refusal through unrecorded.

`fabricengine/doc.go` gains a line naming the mode and `ErrEntryEnded`.

`TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` is tier1 and runs each of its owed rows once more in the mode, as a variant.
Each variant pushes with a run slug and asserts the class its row asserts, `IsPushOwed`,
and that `FailedPushesOwedBy(hub, slug)` and the failed-push directory hold no record:
- the `PrimeRecordsPushOwed_*` rows: `BackoffSpent` returns the deferred class through `recordOwedRecordsPush`,
  and `OriginLookupTransportFailure`, with origin's URL at `refusedOriginURL`, the transport class;
- `Deferred_HeldWeftWriteLock`: the deferred class.

`TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` is tier1 and runs its `TeardownLockHeldPastTheWaitOwesTheRecordsPush` row once more in the mode,
asserting the teardown_lock class, `IsPushOwed` and no record in the same way.

It also gains rows in the mode:
- the weft write lock held across a rejected records push, as a transition commit holds it, then released and a commit made under it:
  the next push in the mode lands, and origin's records branch holds every local commit in its local order
  (the constraint's regression check: a commit made while a replay waits is kept, and origin gets every commit in order);
- a code branch origin diverged from: a not_fast_forward record is written, as outside the mode;
- a records branch deleted on origin whose entry origin's board ended: an error wrapping `ErrEntryEnded`, not owed,
  the records branch's tracking ref is gone, and its `branch.<branch>.merge` stays set;
- a code branch deleted on origin whose entry origin's board ended, with its tracking ref unpruned: the push returns nil,
  the code branch's tracking ref is gone, and its `branch.<branch>.merge` stays set;
  a second push of the pair, after a new records commit, returns nil too, and origin still lacks the code branch.

These rows reach `DeleteRemoteTrackingRef` and the `UpstreamSHA` change through the pair push, so neither gets a gitrepo test of its own.

**ImpactSummary:** Adds an opt-in field to the pair push and the run push, one sentinel on an existing error, a gitrepo method that deletes one tracking ref, and `ErrRefNotFound` on `UpstreamSHA`'s missing-ref error; every existing caller passes the zero value and keeps writing owed records as today.
