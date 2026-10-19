# Card 9 — creel-deliver-marked-probe

**Edit:**
- `internal/testkit/creelfake#Delivery`
- `internal/testkit/creelfake#NewDelivery`
- `internal/testkit/creelfake#Delivery.Paused`
- `internal/creelengine_test#holdAfterAReminderProbeReadTheDraft`
- `internal/creelengine_test#orchStrandIsRemindedAndTypedInto`
- `internal/creelengine_test#TestScheduler_Reminders`
- `internal/creelengine_test#TestScheduler_Injects`

**Create:**
- `plan:internal/testkit/creelfake#Delivery.PausedCalls` -> `func (d *Delivery) PausedCalls(name string) int`
- `plan:internal/creelengine_test#env.awaitPauseRead` -> `func (e *env) awaitPauseRead(t *testing.T, receiver string, n int)`
- `plan:internal/creelengine_test#env.awaitMarkedProbe` -> `func (e *env) awaitMarkedProbe(t *testing.T, receiver string, mark int)`

**Uses:**
- `internal/creelengine_test#env`
- `internal/creelengine_test#env.tick`
- `internal/creelengine_test#env.probeWaits`
- `internal/creelengine_test#awaitTyped`
- `internal/creelengine_test#env.typed`
- `internal/creelengine_test#env.post`

**Intent:** D6, #744: the deliver tests race their own scheduler.
`probeWaits(before + n)` counts any probe that went back to sleep, and a worker's one-slot wake channel can hold a wake left by an earlier post, so a probe that snapshotted the store before the post under test satisfies the count and the test reads `typed` too early, seeing only the owner's notice.
The scheduler is unchanged: a full wake channel already guarantees a probe that will see a post.
The fix is in the tests: they synchronize on a probe that saw the post.

- `creelfake.Delivery` records each `Paused` call per strand, as it records `Idle` calls: a `pausedCalls` map field initialised in `NewDelivery`, incremented in `Paused` before it answers, read by `PausedCalls`.
- The marker: a live receiver's probe reads its pause after the strand read and before it snapshots the store, and no other reader asks about that strand's pause, so a pause read that starts after the post returned belongs to a probe whose snapshot includes the post.
  The store's interrupt path is the one other pause reader (`Store.Interrupt`, and an inject with its interrupt set), so a subtest that interrupts takes its mark after that call returns.
  A test takes `mark := e.del.PausedCalls(receiver)` after the post (or interrupt) returns; the counter counts every `Paused` call from the fake's creation, so a mark and the reads after it are always on one count.
- `awaitPauseRead(t, receiver, n)` polls `PausedCalls(receiver)` until it reaches `n`, under a real-time deadline that only bounds a failing test, as `awaitTyped` does.
- `awaitMarkedProbe(t, receiver, mark)` proves the marked probe ran to its end: it waits for the marked read (`awaitPauseRead(t, receiver, mark+1)`), then advances the clock by the probe ceiling (8 s) and waits for the receiver's next read (`mark+2`).
  A worker's probes are serial, so that next read comes only after the marked probe finished and its worker woke from the wait that followed it.
  The wait is attributed to the receiver this way, never through the global probe-wait count, which every other receiver's worker on the shared fake clock also adds to.
  The 8-second advance stays inside every hold, TTL and back-off a negative check runs under, and a subtest whose later assertion counts probe waits or ticks takes the advance into that count.
- Every subtest in `deliver_test.go` that waits by counting probe waits after a post, directly through `probeWaits(before + n)` or through `tick`, is reshaped by what it then checks:
  a negative check (nothing typed, nothing reminded) takes its mark after the post and calls `awaitMarkedProbe`, advancing the clock past the worker's wait first where the post leaves nothing typed and the probe comes only then, so the check proves the probe under test ran;
  a positive check waits instead for the outcome it asserts, the typed line through `awaitTyped` or the journal event, since an unmarked stale-wake probe that snapshots after the post can deliver it and retire the worker, leaving no marked probe to come.
  `tick` keeps its clock-advance shape for the subtests whose probe is not triggered by a post (back-off and interval checks), and the helpers `holdAfterAReminderProbeReadTheDraft` and `orchStrandIsRemindedAndTypedInto` take the same reshaping.
  This covers the subtests with more than one strand reader, such as `ProgramMailboxIsNeverReminded`, `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` and `LogByIDListsTheRemindedEventsThatHoldTheID`.
  A probe of a receiver that is not live, and a program mailbox, reads no pause and types nothing whatever it snapshots, so a check on such a receiver keeps its wait count.
  `sweepStep` counts sweep waits and is not part of the race; `TestSweep_RemovedHolders` is unchanged.
- Before committing, the implementer runs `lyx gate test ./internal/creelengine -- -run 'TestScheduler_' -count=1000 -cpu=1,2,4` in the foreground and states its result in the commit body, so the deliver subtests are shown to pass at that count.

**Verify:** lyx gate test ./internal/creelengine -- -run 'TestScheduler_' -count=1000 -cpu=1,2,4

**ImpactSummary:** Test-only: the creel fake gains a pause-call counter and the deliver tests re-synchronize on it; no production code changes.
