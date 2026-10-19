# Card 9 — creel-deliver-marked-probe

**Edit:**
- `internal/testkit/creelfake#Delivery`
- `internal/testkit/creelfake#NewDelivery`
- `internal/testkit/creelfake#Delivery.Paused`
- `internal/creelengine_test#TestScheduler_Reminders`
- `internal/creelengine_test#TestScheduler_Injects`

**Create:**
- `plan:internal/testkit/creelfake#Delivery.PausedCalls` -> `func (d *Delivery) PausedCalls(name string) int`
- `plan:internal/creelengine_test#env.awaitMarkedProbe` -> `func (e *env) awaitMarkedProbe(t *testing.T, receiver string, mark, seen int) (advanced time.Duration, passes int)`

**Uses:**
- `internal/creelengine_test#env`
- `internal/creelengine_test#env.tick`
- `internal/creelengine_test#env.probeWaits`
- `internal/creelengine_test#isProbeWait`
- `internal/testkit/creelfake#Clock.Waits`
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
  The wake a post sends runs before the post returns, so the probe it starts can read the pause before the caller reads `mark`; that probe is not the marked one, and the marked probe then comes only after the clock advances past the wait that probe records.
  A test takes `seen`, the probe-wait count (`e.clock.Waits()` filtered by `isProbeWait`), before the post (or interrupt) call, and `mark := e.del.PausedCalls(receiver)` after it returns; the counter counts every `Paused` call from the fake's creation, so a mark and the reads after it are always on one count.
- `awaitMarkedProbe(t, receiver, mark, seen)` proves the marked probe ran to its end, and returns the total clock time it advanced and the passes that advanced it.
  It is one loop under a real-time deadline that only bounds a failing test, as `awaitTyped`'s deadline does, with no wait of its own for the marked read, because the marked read can depend on an advance the loop makes.
  `seen` is taken before the post because the post's wake runs before the post returns, so a probe it starts can record its wait between the post's return and the caller's reads, and a wait counted into `seen` would leave pass 1 waiting for a wait that never comes.
  A wait an earlier, unmarked probe records after `seen` was taken only adds one retried pass.
  Each pass polls both conditions under that deadline: `PausedCalls(receiver)` reaching `mark+2`, which ends the loop with no further advance, and the probe-wait count, read from `e.clock.Waits()` filtered by `isProbeWait`, reaching `seen+pass`, one more than the pass before it, after which the pass advances the clock by the probe ceiling (8 s).
  A blocking `AwaitWaitsWhere` cannot stand in for that poll, since it never watches `PausedCalls`, so a pass would not end when `mark+2` is reached first.
  A worker's probes are serial, so the read at `mark+2` comes only after the marked probe finished and its worker woke from the wait that followed it.
  The worker registers that wait through `clock.After` only after the probe returns, and `Advance` fires only timers already registered, so an advance made before the worker parked is lost; the loop cannot lose it for good, since every advance waits for a newly recorded probe wait, the receiver's own wait is one of them, and the pass that follows its recording fires it (its due time is at most the 8 s ceiling past the clock).
  A pass whose new wait was another receiver's worker's, and whose advance therefore landed before the receiver parked, is retried by the next pass.
  The wait is attributed to the receiver this way, never through the global probe-wait count alone, which every other receiver's worker on the shared fake clock also adds to.
  Bound: in a subtest whose only live worker after the post is the receiver's, the helper makes at most three passes, 24 s, for a single post: one for a probe in flight at the post, one for a wake-driven probe whose pause read came before `mark`, one for the marked probe's own wait.
  The wake channel holds one signal and every post made while a probe runs refills it, so each further post made before the helper is called can add one wake-driven probe before `mark` and so one more pass: `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry`, which posts twice before its first call, is bounded by four passes, 32 s, far under its 30-minute hold.
  A wake left in the worker's channel does not save the marked probe's pass: the worker's `select` evaluates `clock.After(next)` before it picks the wake, so the wait is recorded and the pass may advance once before it sees `mark+2`.
  Each probe wait another live worker records before the receiver parks adds one pass of 8 s.
  So each negative check asserts, after the helper returns, that the total it advanced is shorter than the shortest hold, TTL, back-off or stall freshness that check runs under, which fails the subtest loudly instead of letting an over-long advance pass it vacuously.
  Two passes advance 16 s, which is not shorter than the 16 s `stallFreshness` (`freshStallLocked` in `pool.go` drops a stall only when its age is strictly greater), so the shorter-than assertion already fails at two passes, and a negative check whose shortest window is not longer than its bound cannot take the helper without failing under `-cpu=2,4`; no negative check this card reshapes runs under such a window (the shortest is `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry`'s 30-minute hold), and the assertion fails loudly if a later one does.
  A subtest whose later assertion counts probe waits or ticks takes the passes the helper made into that count.
- Postcondition of the two waits: `awaitMarkedProbe` returns when the receiver's worker has begun the probe after the marked one (the pause read at `mark+2`), and `awaitTyped` returns when the typed line is visible, which the fake records at `Send`, before that probe journals its outcome and registers its next `clock.After`.
  In both cases the worker is not parked, and after the helper's passes its backed-off wait is 2 s to 8 s, not 1 s.
  So a subtest that went through either wait never follows it with a bare `tick(d)` or `probeWaits(before + n)`: `tick` reads `before` and advances, an advance made before the worker registers its wait fires nothing for it, and the worker's own late wait then satisfies the count with the step unseen.
  Each later step that flips a fake or jumps the clock takes `seen` before the flip or jump and `mark` after it, calls `awaitMarkedProbe` again, and then reads `typed` or calls `awaitTyped` for the lines it asserts.
  `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` is the subtest with such a step: it takes `seen`, makes its 31-minute jump with `e.clock.Advance(31 * time.Minute)`, takes `mark`, calls `awaitMarkedProbe` and then `awaitTyped` for the full `want`, and drops the `tick(time.Second)` after the jump: one probe types one item, the notice before the reminder, so the first two probes that begin after the jump, both past the hold's expiry, type the notice and then the reminder; the marked probe is one of those two, so the second has begun by the time the helper returns, and `awaitTyped` waits for its line.
  Where a step needs a parked worker, only a worker whose last recorded wait was awaited by `probeWaits(1)` after `e.start`, by `driveProbeWaits` or by a `tick` that a parked worker entered, with no wake sent and no clock advance past that wait since, qualifies.
- Every subtest in `deliver_test.go` that waits by counting probe waits after a post made once `e.start` has run, directly through `probeWaits(before + n)` or through `tick`, is reshaped by what it then checks:
  a negative check (nothing typed, nothing reminded) takes `seen` before the post and its mark after it, and with several posts takes `seen` before the first and `mark` after the last, and calls `awaitMarkedProbe`, which advances the clock itself where the post leaves nothing typed and the probe comes only then; the caller never advances first, so the check proves the probe under test ran;
  a positive check waits instead for the outcome it asserts, the typed line through `awaitTyped` or the journal event, since an unmarked stale-wake probe that snapshots after the post can deliver it and retire the worker, leaving no marked probe to come.
  `tick` keeps its clock-advance shape for the subtests whose probe is not triggered by a post (back-off and interval checks), and a subtest or helper that posts before `e.start`, such as `holdAfterAReminderProbeReadTheDraft` and `orchStrandIsRemindedAndTypedInto`, is unchanged: `Start` creates the worker with no wake signal and its first probe snapshots every post, so its `probeWaits(1)` wait already waits for a probe that saw the post, and it never calls `awaitMarkedProbe`.
  The same holds for a single post made once `e.start` has run to a receiver that has had nothing pending since `e.start`: `Start` and every earlier wake left that receiver with no worker, so the post's wake creates its worker with an empty wake channel, the worker's first probe snapshots after the post, and the subtest keeps its `probeWaits(1)` and the ticks after it.
  That covers `PostToALiveIdleStrandIsRemindedOnce`, `ReplyToAStrandWithNoWorkerIsRemindedOnce`, the first post of `UnansweredReminderBacksOffToTheCeilingAndANewMessageResets` and of `LogByIDListsTheRemindedEventsThatHoldTheID`, `BusyReceiverIsRemindedAtItsNextIdleEdge` and `PauseTypesNothingUntilLifted`, whose flip and `tick(time.Second)` after `probeWaits(1)` therefore meet a parked worker.
  It does not cover a second post made while the first post's worker may still run, such as `HoldMutes...`'s second post and owner posts, the `fresh` post of `UnansweredReminder...` and the second post of `LogByID...`, which the rule reshapes.
  An inject made once `e.start` has run wakes the worker as a post does (`wakeLocked` in the inject path of `verbs.go`), so it falls under this rule as a post, with the exceptions below.
  `holdAfterAReminderProbeReadTheDraft` being unchanged covers only its own pre-start post, not the `InjectWait` that `StallClearedUnderAHoldIsNotSeededIntoALaterWaiter`, `PersistingStallIsToldOnceToALaterWaiterAndEachChangeOnce` and `ExpiryNamesNoStallLastReadBeforeTheBound` each make after it.
  These three never call `awaitMarkedProbe`, and none waits on a marked probe after its inject.
  In `PersistingStall...` the helper's advance of 8 s or more would move `e.clock.Now()` past the probe that told the waiter, which breaks its `note.At.Equal(e.clock.Now())`.
  `StallCleared...` waits on `waiter.C` through `awaitOutcome`, with the clock unmoved after the inject, and is unchanged.
  `PersistingStall...` keeps its `probeWaits(before + 1)`, because the worker is parked when the inject is made, with no stale wake and no probe in flight: `Start` creates the worker with an empty wake channel, nothing is posted after `e.start`, `Hold` sends no wake, and the last `tick` of `tickPastTheStallFreshness` returned on the worker's newly recorded wait; so the next recorded wait comes from the probe the inject's wake starts, which saw the inject.
  `ExpiryNames...` injects before `tickPastTheStallFreshness`, with the worker parked for the same reasons, and the inject's wake-driven probe can still be running when the first tick reads `before`; a tick that lands one probe ahead can carry that state through every later tick, so the final `e.clock.Advance(time.Minute)` can fire nothing, and since only a probe ages the inject, `awaitOutcome` then times out.
  It takes `before := len(e.probeWaits(0))` before its `InjectWait` and calls `e.probeWaits(before + 1)` after it, so every later tick and the final advance land on a parked worker; its checks need nothing more, since the hold makes every probe skip the pane read and the clock moves a fixed 24 s and a minute past the only probe that read the draft.
  `InjectPostedMidWaitIsChargedFromItsPostingNotFromThePreviousProbe` injects after `driveProbeWaits(4)` and `e.clock.Advance(7 * time.Second)`, which leave the worker parked on its 8 s wait with an empty wake channel, and then ticks; it takes `before := len(e.probeWaits(0))` before its `InjectWait` and calls `e.probeWaits(before + 1)` after it, so the probe the inject's wake starts has parked before its first `tick`, and that tick and the ones after it never advance while a probe is in flight.
  The reshaping by check covers the subtests with more than one strand reader, such as `ProgramMailboxIsNeverReminded`, `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` and `LogByIDListsTheRemindedEventsThatHoldTheID`.
  A probe of a receiver that is not live, and a program mailbox, reads no pause and types nothing whatever it snapshots, so a check on such a receiver keeps its wait count.
  `sweepStep` counts sweep waits and is not part of the race; `TestSweep_RemovedHolders` is unchanged.
- Before committing, the implementer runs `lyx gate test ./internal/creelengine -- -run 'TestScheduler_' -count=1000 -cpu=1,2,4` in the foreground and states its result in the commit body, so the deliver subtests are shown to pass at that count.

**Verify:** lyx gate test ./internal/creelengine -- -run 'TestScheduler_' -count=1000 -cpu=1,2,4

**ImpactSummary:** Test-only: the creel fake gains a pause-call counter and the deliver tests re-synchronize on it; no production code changes.
