# Card 10 — teardown-pushes-first

**Edit:**
- `internal/battenshed#TeardownDeps`
- `internal/battenshed#worktreeTeardownProducer.Call`
- `internal/battenshed#NewWorktreeTeardown`
- `internal/battenshed/doc.go`
- `internal/battencli#battenCLI.wire`
- `internal/battenshed#teardownCallRecorder.deps`
- `internal/battenshed#TestWorktreeTeardown_TearsDownInOrderOnceTheLockIsFree`
- `internal/battenshed#TestWorktreeTeardown_ShutdownFailureNeverCallsRemove`
- `internal/battenshed#TestWorktreeTeardown_PrimeLockDispositions`
- `internal/battencli#TestBattenIntegration_Rows`
- `internal/battencli#TestWire_BuildsLazilyForANonexistentTaskWorktree`
- `internal/testkit/envkit#nilLegal`

**Create:**
- `plan:internal/battenshed#recordingPrimeLock` -> `func recordingPrimeLock(rec *teardownCallRecorder) PrimeLock`

**Uses:**
- `internal/runpusher#PushUntilLanded`
- `internal/fabricengine#TipsOnOrigin`
- `internal/fabricengine#ErrEntryEnded`
- `internal/pairteardown#PushRun`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:**
Make Worktree-Teardown push the pair to completion before it removes it, the one row that waits for a push (D5).
Today it removes the pair under the prime lock with no push of its own, trusting each transition's push.

- `TeardownDeps` gains `Push func(ctx context.Context) error` and `Level func() (bool, error)`.
  A nil `Push` pushes nothing and a nil `Level` reads level, so a hand-built receiver keeps today's sequence.
  `envkit`'s `nilLegal` gains `Teardown.Push` and `Teardown.Level`, since nil is their documented default.
  `NilSeams` then skips both, so loomcli's `intentionallyNil` stays as it is:
  an entry there for a path `NilSeams` skips fails `TestWire_EverySeamFilled`.
- `worktreeTeardownProducer.Call` runs, in order:
  take the prime lock; shut the session down; release the lock;
  `Push`, with no prime lock held, so the owed backoff never holds it;
  take the prime lock again; `Level`; when it reads false, a tip moved since the push, so release the lock and push again;
  when it reads true, `Remove`, then release.
  A `Push` error wrapping `fabricengine.ErrEntryEnded` means origin ended the pair's records branch and nothing is left to push:
  the producer takes the lock and removes without reading `Level`.
  A pair whose code branch origin deleted while its entry ended pushes with no error and loses that branch's tracking ref (card 1),
  so `Level` reads it held (card 2) and the removal goes on for such a pair.
  A `Push` error wrapping `fabricengine.ErrHandedOver` is Stuck naming the hand-over:
  the hand-over already removed the pair and the run belongs to the other machine, so no resume here applies.
  Any other failed `Push`, a refused push, is Stuck with the push error, whose text carries its record's way forward,
  and names the resume `lym batten run <slug>` once that way forward is done; the pair is kept and `Remove` is not called.
  A failed `Level` read is Stuck naming it and the same resume.
  A cancelled context returns its error, as today's cancellation paths do.
  `NewWorktreeTeardown`'s doc states the order.
- `battenCLI.wire` fills both for the run's pair:
  `Push` runs `runpusher.PushUntilLanded` over `pairteardown.PushRun` with `OwedUnrecorded`, `StatusPushLockWait` for both locks,
  `fabricengine.IsPushOwed`, the real sleep and no ceiling notice;
  a pair already gone pushes nothing and returns nil, so a resumed row whose pair is gone goes on to its removal as today.
  `Level` is `fabricengine.TipsOnOrigin` on the pair, true for a pair already gone.
  The archive push in the removal stays as it is.
- `battenshed/doc.go`'s Worktree-Teardown lines state the push between the two prime-lock holds.

`recordingPrimeLock` appends "lock" and "release" to the recorder's calls, and `teardownCallRecorder.deps` gains recording `Push` and `Level` seams, level on the first read.
Tests, all tier0 unless named:
- `TestWorktreeTeardown_TearsDownInOrderOnceTheLockIsFree` asserts the calls lock, shutdown, release, push, lock, level, remove, release;
  a row with `Level` false once then true records two pushes and one remove.
- `TestWorktreeTeardown_ShutdownFailureNeverCallsRemove` gains rows: a refused push is Stuck with the push error and the resume, and `Remove` is never called;
  a handed-over push is Stuck naming the hand-over; a failed `Level` is Stuck; a push error wrapping `ErrEntryEnded` removes without reading `Level`.
- `TestWorktreeTeardown_PrimeLockDispositions` gains a row: a Stuck push still released the first hold and takes no second.
- `TestBattenIntegration_Rows` is tier1 and gains a row: a resumed Worktree-Teardown whose pair is already gone finishes Done.
- `TestWire_BuildsLazilyForANonexistentTaskWorktree` gains the rows `TeardownPush` and `TeardownLevel`, asserting `battenCLI.wire` fills both,
  since `nilLegal` keeps `TestWire_EverySeamFilled` from checking them.

**ImpactSummary:** One producer's sequence and its two new seams, wired in batten's wire; no other row changes.
