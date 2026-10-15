# Card 9 — guid-engine-reads

**Edit:**
- `internal/shuttleengine#Runner.InterruptSettle` -> `func (r *Runner) InterruptSettle(guid string) (time.Duration, error)`
- `internal/shuttleengine#Runner.ContextTokens` -> `func (r *Runner) ContextTokens(guid string, turnEnd Event) (ContextReading, error)`
- `internal/shuttleengine#Runner.CompactedSince` -> `func (r *Runner) CompactedSince(guid string, turnEnd Event, since time.Time) (CompactionBoundary, bool, error)`
- `internal/shuttleengine#Runner.ClassifySkillLoad` -> `func (r *Runner) ClassifySkillLoad(guid string, turnEnd Event, skills []string) (SkillLoadReport, error)`
- `internal/shuttleengine#TestRun_Interrupt_PlaysEscape`
- `internal/shuttleengine#TestRunner_SessionMethods_ErrorOnPlainEngine`
- `internal/shuttleengine#TestRunner_LoadSkillsAndClassifySkillLoad_DriveTheEngine`
- `internal/shuttleengine#TestRunner_LoadSkillsAndClassifySkillLoad_ErrorOnPlainEngine`
- `internal/shuttleengine/enginedispatch_test.go`
- `internal/orchcli#runnerSession.InterruptSettle` -> `func (s runnerSession) InterruptSettle(guid string) (time.Duration, error)`
- `internal/orchcli#runnerSession.ContextTokens` -> `func (s runnerSession) ContextTokens(guid string, turnEnd shuttleengine.Event) (shuttleengine.ContextReading, error)`
- `internal/orchcli#runnerSession.CompactedSince` -> `func (s runnerSession) CompactedSince(guid string, turnEnd shuttleengine.Event, since time.Time) (shuttleengine.CompactionBoundary, bool, error)`
- `internal/orchcli#runnerSession.ClassifySkillLoad` -> `func (s runnerSession) ClassifySkillLoad(guid string, turnEnd shuttleengine.Event, skills []string) (shuttleengine.SkillLoadReport, error)`
- `internal/orchengine#Session.InterruptSettle`
- `internal/orchengine#Session.ContextTokens`
- `internal/orchengine#Session.CompactedSince`
- `internal/orchengine#Session.ClassifySkillLoad`
- `internal/orchengine#Watcher.tick`
- `internal/orchengine#Watcher.tickIdle`
- `internal/orchengine#Watcher.settleSkillTurn`
- `internal/orchengine#Watcher.tickResuming`
- `internal/orchengine#Watcher.tickCompacting`
- `internal/orchengine#Watcher.handleHubEntry`
- `internal/orchengine#fakeSession`
- `internal/orchengine#fakeSession.InterruptSettle` -> `func (f *fakeSession) InterruptSettle(guid string) (time.Duration, error)`
- `internal/orchengine#fakeSession.ContextTokens` -> `func (f *fakeSession) ContextTokens(guid string, ev shuttleengine.Event) (shuttleengine.ContextReading, error)`
- `internal/orchengine#fakeSession.CompactedSince` -> `func (f *fakeSession) CompactedSince(guid string, turnEnd shuttleengine.Event, since time.Time) (shuttleengine.CompactionBoundary, bool, error)`
- `internal/orchengine#fakeSession.ClassifySkillLoad` -> `func (f *fakeSession) ClassifySkillLoad(guid string, turnEnd shuttleengine.Event, skills []string) (shuttleengine.SkillLoadReport, error)`
- `internal/orchengine#TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`
- `internal/tenderengine#Pair.InterruptSettle`
- `internal/tenderengine#settle`
- `internal/tenderengine#fakePair`
- `internal/tenderengine#fakePair.InterruptSettle` -> `func (p *fakePair) InterruptSettle(guid string) (time.Duration, error)`
- `internal/tenderengine#TestPause`
- `internal/tendercli#shuttlePair.InterruptSettle` -> `func (p shuttlePair) InterruptSettle(guid string) (time.Duration, error)`

**Intent:** Give the four `Runner` methods that consult an engine with neither a guid nor a run the guid of the run they read, so they resolve its recorded engine (D4), and carry the guid through the orch watcher's and tender's session seams.

`InterruptSettle`, `ContextTokens`, `CompactedSince` and `ClassifySkillLoad` each take `guid` first, read the run with `FindRun`, and resolve its engine with `engineFor`.
A guid that is not a shuttle strand fails with the `is not a shuttle strand` error the other guid methods return, and a record naming an unregistered engine fails with the lookup's error.
`InterruptSettle` now returns an error for an unresolvable record; it still answers 0 for an engine without `InterruptGuard`.
The capability errors (`SessionCycler`, `SkillLoader`) are unchanged and now name the recorded engine's lack.

Callers hold the guid (D4):
- `orchengine.Session`'s four methods take `guid` first, with their doc comments saying so, and `runnerSession` passes it through.
  The watcher passes `st.Strand`, the orch run's strand guid, in `tick`, `tickIdle`, `settleSkillTurn`, `tickResuming` and `tickCompacting`.
  `handleHubEntry` logs a failed `InterruptSettle` at Warn, naming the strand and the error, and waits no settle, since the orch is its one session and none resolved.
- `tenderengine.Pair.InterruptSettle` takes `guid` and returns an error, and `shuttlePair` passes it to its runner.
  `settle` asks each keyed session's pair with its guid, logs each failure at Warn naming the strand and the error, and waits the longest settle among the sessions it did resolve, none when every resolution fails.

Tests:
- `TestRun_Interrupt_PlaysEscape` reads `InterruptSettle` with its run's guid.
- `TestRunner_SessionMethods_ErrorOnPlainEngine` and both `TestRunner_LoadSkillsAndClassifySkillLoad_*` tests pass the guid of a seeded run record (`withStrand`).
- `TestRunner_EngineSetDispatch` in `enginedispatch_test.go` gains rows over its `dispatchEngine`s: after a `Start` on `b`, `InterruptSettle(guid)` answers `b`'s settle, distinct from `a`'s, and `ContextTokens`, `CompactedSince` and `ClassifySkillLoad` with the guid consult `b` and never `a`; each of the four on a record naming `c` returns an error wrapping `ErrEngineNotRegistered`, and each on a guid with no run record fails with the `is not a shuttle strand` error.
- `fakeSession` gains `settleErr error`, returned by its `InterruptSettle`; `TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine` gains a row: with `settleErr` set, the first tick after the first key sends the second key to a still-busy orch without waiting a settle, and the captured log holds one Warn line naming the strand.
- `fakePair` gains `settleErr error`, returned by its `InterruptSettle`; `TestPause` gains a row: of two busy pairs, one with a 2 s settle and one whose settle fails, the pause sleeps exactly 2 s once per pass and the captured log holds a Warn line naming the failing strand, and with both settles failing it sleeps zero.

**ImpactSummary:** Four exported Runner methods re-signed; the cascade reaches orchcli's and tendercli's adapters, the orchengine Session and tenderengine Pair interfaces with their in-package callers and fakes, and five tests.
