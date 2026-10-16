# Card 2 — watcher-records-session-id

**Create:**
- `internal/orchengine#Watcher.recordSessionID`
- `internal/orchengine#TestWatcher_RecordsTheSessionIDItReads`

**Edit:**
- `internal/orchengine#State`
- `internal/orchengine#ResetForFreshLaunch`
- `internal/orchengine#Session`
- `internal/orchengine#Watcher.tick`
- `internal/orchengine#Watcher.logIdleStateDisagreement`
- `internal/orchengine#fakeSession`
- `internal/orchengine#fakeSession.SessionState`
- `internal/orchengine#TestWatcher_StrandReboundMidTickIsNotOverwritten`
- `internal/orchengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/orchengine#newWatchEnv`
- `internal/orchengine#watchEnv.tick`
- `internal/orchengine#watchEnv.state`
- `internal/orchengine#watchEnv.setState`

**Intent:** Orch's persisted state records the provider session id of the strand it names, and the watcher keeps it current across `/clear` (D4).

- `State` gains `SessionID string` with JSON key `session_id`, commented as the provider session id of the session `Strand` runs: written by `start` at every fresh launch, cleared when `start` binds another live strand, and kept current by the watcher.
  The field is optional on read (`PATTERN-tolerant-records`): a state file an older binary wrote decodes with it empty.
- `ResetForFreshLaunch` leaves `SessionID` untouched; its doc comment's survivor list gains `SessionID`, "which the caller sets for the session it launched or bound".
  The watcher's `initCursor` reset of a phase from another strand keeps the id, since the strand is unchanged.
- `Watcher.recordSessionID` takes shuttle's files-only reading through `w.session.SessionState(st.Strand)`, whose `SessionID` is folded over all of the run's signals (card 1).
  A read error is logged at Debug and returns nil, so an unreadable reading, or a bound strand with no shuttle run, leaves the recorded id and never fails the tick.
  An empty reading id, or one equal to `st.SessionID`, writes nothing.
  Otherwise it logs at Info (`orch: session id recorded`, with the strand guid, the previous and the new id), sets `st.SessionID` and saves through `w.save`, the strand-checked `saveStateForStrand`, so a watcher never writes onto a strand a concurrent `start` bound; that save's `errStrandReplaced` is returned for `Tick` to absorb as it does today.
  It accepts whatever id the newest id-naming signal of the strand's own run names, bounded to that run's events file.
  That rests on the provider naming the parent session id in a subagent's or fork's hook lines, since the orch strand runs with `ForkSubagents` and the claude engine takes each hook line's `session_id` as given, so a provider that names the subagent's own id there records the subagent's session, which `--continue` then resumes.
- `Watcher.tick` calls `recordSessionID(&st)` once per tick, after the strand-alive check and the `initCursor` binding and before `ReadEvents`, returning its error.
  Compact mode keeps one id, so after a launch it writes nothing there.
- The `Session` interface's `SessionState` method comment says the watcher also records the reading's session id, and `logIdleStateDisagreement`'s doc drops "dropping a conditioned notice is the one decision the watcher takes from the state", naming the two decisions: dropping a conditioned notice and recording the session id.

Tests:

- `fakeSession` gains `sessionID string`, answered as `RunSessionState.SessionID`; the `sessionSinces` comment says the slice is consumed one per `SessionState` call, which the per-tick read makes one more per tick.
  `TestWatcher_LogsDisagreementBetweenIdleProbeAndSessionState` needs no change: with the per-tick read and the probe each reading once per tick, its "a disagreement with a new since logs again" script `{t0, t0, t0.Add(time.Minute)}` still logs on the first and second ticks only, and its unreadable drive still matches the readable one, since the fake answers an empty id and the per-tick read swallows the error.
- `TestWatcher_RecordsTheSessionIDItReads` is table-driven over `newWatchEnv` (strand `s1`), seeding `State.SessionID` with `setState` and ticking once with `Tick`, asserting the tick's error and the saved state:
  - "a reading naming a new id records it": recorded `launch-id`, fake `post-clear-id`; want `post-clear-id`.
  - "an unchanged id writes nothing": recorded and fake `same-id`; the test sets the state file's mtime to an hour back with `os.Chtimes` before the tick, and wants the mtime unchanged and the id `same-id`.
    This tick has no events, no notice and a typed color, so nothing else saves; the row ticks once before setting the mtime to type the start color.
  - "a reading naming no id keeps the recorded one": recorded `post-clear-id`, fake empty; want `post-clear-id`.
  - "an unreadable reading keeps the id and the tick goes on": recorded `launch-id`, `sessionStateErr` set; want a nil error and `launch-id`.
- `TestWatcher_StrandReboundMidTickIsNotOverwritten` gains a second case, "a session id read for a rebound strand is not written": state `s1` recording `launch-id`, fake alive with `sessionID` `other-id`, and `onAlive` rebinding the state to strand `s2` with an empty id through `setState`.
  The tick loaded `s1`, `initCursor` binds idle without saving, and `recordSessionID`'s save meets the rebind; want `Tick` to return false and nil, strand `s2` and an empty id.
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/orchengine` by one for the new top-level test.

`internal/orchengine/doc.go`'s `# The watcher` section gains that each tick reads the strand's session id from shuttle's reading and records it in State when it changes, through the strand-checked save, and its `# The .lyx/orch/ layout` entry for `state.json` notes that the state carries the orch session's id and outlives `lyx orch stop`.
Its `# Open risks` section gains the provider reliance above: the recorded id is right only while the provider names the parent session id in a subagent's or fork's hook lines.

**ImpactSummary:** One new State field read by nobody but `start`, `status` and the watcher; one extra files-only reading per tick, whose error never fails a tick; no existing test's assertions change, and two interface and method comments are reworded.
