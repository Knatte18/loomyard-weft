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
- `internal/pairteardown#RunPush`
- `internal/pairteardown#PushRun`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`

**Extends:**
- `internal/fabricengine/doc.go`

**Create:**
- `plan:internal/fabricengine#ErrEntryEnded` -> `var ErrEntryEnded error`
- `plan:internal/fabricengine_test#TestPushPairAnchored_OwedUnrecorded` -> `func TestPushPairAnchored_OwedUnrecorded(t *testing.T)`

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
- `pairteardown.RunPush` gains `OwedUnrecorded`, and `PushRun` passes it into `PairPush`.

So `FailedPushesOwedBy` returns no record of a push made in the mode,
and the owed-push read in `internal/pairteardown/runread.go` leaves `PushOwed` unset (D8):
the act rule's readers keep their behaviour and never see the pusher's retries.
`IsPushOwed` still reads every returned owed error as owed, since it reads the error's class, not the record.
The mode skips only the record of a failure that is retried anyway; it lets no refusal through unrecorded.

`fabricengine/doc.go` gains a line naming the mode and `ErrEntryEnded`.

`TestPushPairAnchored_OwedUnrecorded` is tier1, in the external `fabricengine_test` package, built on `newDivergedPair`.
Each row pushes in the mode with a run slug and asserts the returned error's class, `IsPushOwed`,
and that `FailedPushesOwedBy(hub, slug)` and the failed-push directory hold no record:
- origin's URL set to `refusedOriginURL`, as the existing transport rows set it: a transport error, owed, no record;
- the pair's teardown lock held exclusive with a positive `TeardownWait`: a teardown_lock error, owed, no record;
- the records worktree's weft write lock held across a rejected records push, as a transition commit holds it:
  a deferred-class error, owed, no record;
  once the lock is released and a commit made under it, the next push in the mode lands,
  and origin's records branch holds every local commit in its local order
  (the constraint's regression check: a commit made while a replay waits is kept, and origin gets every commit in order);
- a code branch origin diverged from: a not_fast_forward record is written, as outside the mode;
- a records branch deleted on origin whose entry origin's board ended: an error wrapping `ErrEntryEnded`, not owed.

`TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` is tier1 and runs its `BackoffSpent` row once more in the mode:
the records side still unpushed past its backoff returns the deferred class through `recordOwedRecordsPush` and writes no record.

**ImpactSummary:** Adds an opt-in field to the pair push and the run push and one sentinel on an existing error; every existing caller passes the zero value and keeps writing owed records as today.
