# Card 19 — batten-liveness-hold-retry

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#innerRunProducer.driverAtWork`
- `internal/battenshed#innerRunProducer.driverAlive`
- `internal/battenshed#innerRunProducer.arbiterHolds`
- `internal/battenshed#innerRunProducer.stateChangeReady`
- `internal/battenshed#innerRunProducer.publishRetryDecision`
- `internal/battenshed#TestInnerRun_AdoptsARunningChildWithADriver`
- `internal/battenshed#TestNotice_DeadDriverWhileRunning`
- `internal/battenshed#TestNotice_LiveArbiterHoldsTheStateChange`
- `internal/battenshed#TestInnerRun_PublishRetryDeclinesRecordsThatDoNotMatchTheHalt`
- `internal/battenshed#noticeHarness`
- `internal/battenshed#noticeHarness.producer`
- `internal/battenshed#retryHarness`
- `internal/battenshed#newRetryHarness`
- `internal/battenshed/doc.go`

**Intent:**
D2, D4 and D7 at batten's Run-Shed wait: a held driver lock reads as a live driver, a tackler in flight holds the stop notice as an arbiter does, and a Go verify retry spends batten's one retry.

`InnerRunDeps` gains three seams, each resolved on Call and each defaulted in `NewInnerRun` when nil:
- `ChildDriverLockHeld func() (bool, error)`: whether the child's Go driver lock is held; nil reports false.
- `TacklerInFlight func(ctx context.Context) (ChildArbiter, bool, error)`: the child's tackler marker read as the arbiter's is, with a deadline and whether its strand is live; nil reports none.
- `GoVerifyRetrySpent func() (bool, error)`: whether the child's settle records show a Go verify retry; nil reports false.

Behavior:
- `driverAtWork` adds a held driver lock to a live or retiring strand and a held run lock, so batten adopts a running go child whose driver settles between steps; a read error is a hard error, as the run lock's is.
- `driverAlive` reads alive when the strand is live, or when the child's driver lock or run lock is held, so a running go child with neither lock and no strand is noticed as driver-dead, and one whose Go driver holds its lock is not.
  A lock read error reads as alive with one Warn, as a failed strand read does.
- `arbiterHolds` also holds while `TacklerInFlight` reports a live tackler within its deadline plus `noticeReportWait`.
  Its doc comment and `stateChangeReady`'s name the tackler.
- In `publishRetryDecision`, with no publish-retry marker, a blocked child at a verify row whose `GoVerifyRetrySpent` reports true returns a spent retry, so batten never retries a run whose Go driver already did (D4).
  A read error is warned about and leaves the halt to the usual handling.

`doc.go` names the driver lock as a liveness signal, the tackler hold, and the retry split between the Go driver's settle records and batten's marker.

Tests:
- `TestInnerRun_AdoptsARunningChildWithADriver` gains a `driverLockHeld` column and a row `HeldDriverLockIsAdopted` (no strand, free run lock), adopting with no spawn.
- `noticeHarness` gains `driverLock` and `tackler`/`tacklerFound` fields, which `noticeHarness.producer` wires into the two new seams.
- `TestNotice_DeadDriverWhileRunning` gains a subtest where no strand is live but the driver lock is held, which sends no driver-dead notice, and keeps the base case of no strand and no lock sending one.
- `TestNotice_LiveArbiterHoldsTheStateChange` gains a row whose live tackler within its deadline holds the notice and one whose tackler's deadline plus the report wait has passed, which sends it.
- `retryHarness` gains `goRetrySpent`, which `newRetryHarness` wires into `GoVerifyRetrySpent`.
- `TestInnerRun_PublishRetryDeclinesRecordsThatDoNotMatchTheHalt` gains a row `GoDriverRetryAlreadySpent` with a matching record, expecting no goto.

**ImpactSummary:** three optional seams on batten's InnerRun deps, each defaulting to today's behavior, read in the adoption, notice and publish-retry paths; battencli wires them in card 21.
