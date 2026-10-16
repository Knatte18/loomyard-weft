# Card 3 — stall-freshness

**Edit:**
- `internal/creelengine#stallNote`
- `internal/creelengine#Store`
- `internal/creelengine#Outcome`
- `internal/creelengine#StallNote`
- `internal/creelengine#Waiter`
- `internal/creelengine#Store.noteStall`
- `internal/creelengine#Store.addWaiterLocked`
- `internal/creelengine#Store.settleInjectLocked`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelengine_test#TestDaemon_Protocol`
- `internal/creelcli#TestCreelCLI`
- `internal/creelengine/doc.go`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`

**Create:**
- `plan:internal/creelengine#stallFreshness` -> `const stallFreshness = 2 * probeBackoffCeiling`
- `plan:internal/creelengine#Store.freshStallLocked` -> `func (s *Store) freshStallLocked(receiver string) (stallNote, bool)`
- `plan:internal/testkit/creelfake#Clock.AdvanceByWaitsWhere` -> `func (c *Clock) AdvanceByWaitsWhere(keep func(time.Duration) bool, done <-chan struct{})`

**Uses:**
- `internal/creelengine#probeBackoffCeiling`
- `internal/creelengine#Waiter.notify`
- `internal/creelengine_test#env.driveProbeWaits`
- `internal/creelengine_test#awaitOutcome`
- `internal/creelengine_test#newRig`
- `internal/testkit/creelfake#Clock.AwaitWaitsWhere`

**Intent:** Fix #682 (D3): a stall reaches a reader only from a fresh reading.

`stallNote` gains `readAt`, the clock time a probe last read the stall, beside `at`, the time the stall began, which stays the time every note and answer prints.
`Store.noteStall` sets `readAt` on every non-empty reading, a repeat of the standing stall included; a changed stall still sets `at`, journals one `stalled` record and notifies, as today.
`Waiter` gains the stall it was last told, and on every non-empty reading `noteStall` notifies each waiter on the receiver's queued injects whose last told stall differs from the reading, and records it as told.
So a waiter seeded with no stall hears a stall that persists at the next reading, once, and a waiter hears each change it sees once: A, then B, then A again reaches it three times.
An empty reading clears the standing stall with no record, as today, and resets each such waiter's last told stall, so a stall that returns after a clear is told again.
`stallFreshness` is twice the probe backoff ceiling, 16 seconds.
`Store.freshStallLocked` returns the receiver's standing stall only when its `readAt` is no older than `stallFreshness` on the store's clock; a stale one is dropped from the standing map, so the next reading of it journals a new `stalled` record, and nothing is returned.
`Store.addWaiterLocked` seeds a new inject waiter, and records it as told, only from `freshStallLocked`, and `Store.settleInjectLocked` takes an expired inject's answer stall, its `StallAt` and its `expired` record's reason from it, so a stall last read before the bound reaches no reader.
Rationale: a probe skips the pane read while a hold keeps a receiver's injects back or nothing is due, so the standing stall is never cleared then.
Bound: a probe runs at least every 8 seconds while it reads, so a live reading is always fresh, and only a stall no probe has read for 16 seconds is withheld.

Tests, rows of `TestScheduler_Injects` on the fake delivery and clock; each reaches the probe skip by a hold of `ly:BATTEN` on alpha after a reminder probe read the draft, then moves the clock past 16 seconds with no read:
- "StallClearedUnderAHoldIsNotSeededIntoALaterWaiter": the draft is cleared in the fake, `InjectWait` from the hold owner starts, its progress channel holds no note when the inject is typed, and its outcome is injected.
- "PersistingStallIsToldOnceToALaterWaiterAndEachChangeOnce": the draft stays, the owner's `InjectWait` gets no seed, the next probe tells it the draft once, a repeat probe tells it nothing, and too-short then draft again are each told once, the channel drained after each probe; then a reading with no reason tells nothing, and the draft read after it is told again; the notes' times are each stall's start.
- "ExpiryNamesNoStallLastReadBeforeTheBound": a non-owner `InjectWait` with a one-minute TTL expires under the hold with an empty `Stall` and zero `StallAt`, and the `expired` log entry has no reason.
The existing rows that read a stall and expire move the clock in steps a probe reads at.
`Clock.AdvanceByWaitsWhere`, in the shared kit both packages' rigs drive, advances the clock by the last recorded wait `keep` accepts, then waits for the next such wait to be recorded before the next step, until `done` closes; like `Clock.AwaitWaitsWhere`, it gives up after a real ten seconds, so a failing test fails on its own assertion instead of hanging.
It never jumps to a pending timer, so the sweep timer a queued inject arms is never fired between two probe readings, as `Clock.AutoAdvance` would fire it.
Each row's call goroutine closes a `returned` channel after it sends its result, and the row runs the daemon clock's `AdvanceByWaitsWhere(isProbeWait, returned)` before it reads the result, so a probe reads at each of its waits until the call returns, and the rows "InjectWaitHearsEachStallChangeAndExpiresWithTheLastOne" and "ResumedInjectWaitHearsTheStandingStallAtOnce" of `TestDaemon_Protocol` and "a waiting inject hears the stall on stderr and an expired answer carries it" of `TestCreelCLI`, which today jump 61 seconds in one advance past the last reading, keep their asserted expired stall through it.

Comments in the same commit: the `stalls` field of `Store` names the stall's start and last read times, `Outcome` says its stall is the receiver's fresh stall at expiry, and `StallNote` says its time is when the stall began.
Docs in the same commit: the Daemon, Inject, Log and Delivery loop paragraphs of `internal/creelengine/doc.go` say a waiter and an expired answer get a stall only from a reading no older than twice the probe ceiling, that a waiter is told each change it sees once, that the printed time is when the stall began, and that a stall read again after it went stale journals a new `stalled` record; the C8 watch line of `tools/sandbox/SANDBOX-CREEL-SUITE.md` adds that an `inject --wait` started after a draft was cleared under a hold prints no stall line and ends injected.

**ImpactSummary:** The stall state behind `inject --wait` progress, seeding and expiry in `internal/creelengine` gains a read time and per-waiter told state; no exported signature changes, the shared `creelfake` kit gains one clock-stepping method, and three existing rows in two test files change only how they step the clock.
