# Card 12 — A run record that reads relaunching, and one running predicate

**Create:**
- `plan:internal/shuttleengine#relaunchBound` -> `func relaunchBound(cfg Config) time.Duration`
- `plan:internal/shuttleengine#RunState.relaunching` -> `func (rs RunState) relaunching(now time.Time, bound time.Duration) (within, past bool)`
- `plan:internal/shuttleengine#updateRunState` -> `func updateRunState(runDir string, mutate func(rs RunState, found bool) (RunState, error)) error`
- `plan:internal/shuttleengine#TestRunState_Relaunching` -> `func TestRunState_Relaunching(t *testing.T)`
- `plan:internal/shuttleengine#TestStopStrand_RelaunchingRecordStopsAndRemovesTheHeldStrand` -> `func TestStopStrand_RelaunchingRecordStopsAndRemovesTheHeldStrand(t *testing.T)`

**Extends:**
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#AgentActivity`
- `internal/shuttleengine#RunSessionState`

**Edit:**
- `internal/shuttleengine/doc.go`
- `internal/shuttlecli#shuttleCLI.stateCmd`
- `internal/shuttlecli#TestStateCmd_ListsRunningRunsAndNarrowsByStrand`
- `internal/shuttleengine#ReadAgentActivity`
- `internal/shuttleengine#runIsLive` -> `func runIsLive(rs RunState, now time.Time, bound time.Duration) bool`
- `internal/shuttleengine#noteWaiterlessRun`
- `internal/shuttleengine#ReadAgentWaits`
- `internal/shuttleengine#unfinished` -> `func unfinished(rs RunState, now time.Time, bound time.Duration) bool`
- `internal/shuttleengine#Runner.carriedInstructions`
- `internal/shuttleengine#ReadSessionStates`
- `internal/shuttleengine#Runner.StopStepRuns`
- `internal/shuttleengine#Run.Stop`
- `internal/shuttleengine#Runner.stopRecorded`
- `internal/shuttleengine#strandRemovable`
- `internal/shuttleengine#dispositionCandidate`
- `internal/shuttleengine#sweepOrphans`
- `internal/shuttleengine#TestReadAgentActivity`
- `internal/shuttleengine#TestReadSessionStates`
- `internal/shuttleengine#TestStart_CarriesOverOperatorInstructions`
- `internal/shuttleengine#TestRunner_StopStepRuns_StopsOnlyTheStepsRuns`
- `internal/shuttleengine#TestAttach_OutcomeDisposition`
- `internal/shuttleengine#seedAttachRunOpts`
- `internal/shuttleengine#TestReadAgentWaits`
- `internal/shuttleengine#TestSweepOrphans_AgeGuardAndLiveGuid`
- `internal/shuttleengine#TestCompletionSignal_NegativeVerdictReturnSites`
- `internal/shuttleengine#TestCompletionSignal_FileContractCallSites`

**Uses:**
- `internal/state#UpdateJSON`
- `internal/shuttleengine#loadRunState`
- `internal/shuttleengine#isTerminalOutcome`

**Intent:**
D7's record half, before any relaunch exists.
`RunState` gains `RelaunchingAt string` (`relaunchingAt,omitempty`), the RFC 3339 time of the mark; the record's `Outcome` stays `runOutcomeRunning` throughout, so a relaunching record is a running record to every reader that compares the outcome.
`relaunchBound` is twice `StartupTimeoutS`.
`RunState.relaunching` is the one predicate: `within` while the mark is set and younger than the bound, `past` once it is older.
Every reader below takes it through that predicate:

- `runIsLive` reads a record within its bound as live whatever its pid, and one past its bound as a running run whose waiter is gone; `noteWaiterlessRun` and `ReadAgentActivity` follow it.
- `unfinished` reads a record within its bound as not unfinished, so a sibling `Start`'s carry-over read never treats it as a dead predecessor.
- `ReadAgentActivity` and `ReadSessionStates` report a relaunching record with the new `AgentActivity.Relaunching` and `RunSessionState.Relaunching` set and no turn-end facts: no API error, no usage limit, no session state from the events file, since its prompt offset precedes the limited turn end until card 14's step 4.
  `lym shuttle state`'s `stateCmd` adds `relaunching` from `RunSessionState.Relaunching` to each run's map.
- `StopStepRuns` stops a relaunching run of its step, within its bound or past it, through `stopRecorded`, whatever its pane state, skipping the held-session branch.
- `strandRemovable` reads a held strand (card 11) as removable, so the stop verb removes a strand held for a relaunch whatever its pane state.
  Its other two callers reach a held strand too: `removeSupersededStrands` removes it through `stopRecorded` for a superseded attach candidate, and `StopStepRuns`' untied branch reports it untied, as intended for a relaunch whose run a fresh run supersedes.
- `dispositionCandidate` reads a relaunching record within its bound over a tracked strand as attachable, whatever its pane state; card 15 handles one past its bound.
- `sweepOrphans` keeps a run directory whose record reads relaunching within its bound, whatever its strand; past its bound the sweep's rule applies as today, so a record no attach settles goes once its strand is gone.

`updateRunState` is one `state.UpdateJSON` hold of `run.json`'s lock.
Both stop forms change to it: `Run.Stop` and `Runner.stopRecorded` read and write the record in one hold, leaving a terminal record as it is, so a stop racing a relaunch write never loses either; `Run.Stop` keeps its handle's copy in step under `recordMu`.
The completion-signal tripwire's pinned site sets are updated for the moved sites in `stop.go` and `attach.go`; no negative verdict is added.

`shuttleengine/doc.go`'s sentences these readers falsify are rewritten: the step stop's rule that a strand is removed when live or a not-live own-window one, and only the record stopped otherwise, gains the held strand as removable; the held-session branch for an untracked or paneless strand gains the relaunching run's exception; and the carry-over's unfinished predecessor reading running with its waiter gone excludes a relaunching record within its bound.

Tests are tier0:
`TestRunState_Relaunching` rows: no mark; a mark inside the bound; a mark past it; an unparsable mark reads past.
`TestReadAgentActivity` and `TestReadSessionStates` gain a relaunching record within its bound whose events file ends on a usage-limit turn end: it is listed, marked relaunching, with no usage limit, and not waiter-gone; past its bound with a dead pid it reads waiter-gone.
`TestStart_CarriesOverOperatorInstructions` gains a row: a predecessor relaunching within its bound carries nothing.
`TestRunner_StopStepRuns_StopsOnlyTheStepsRuns` gains a relaunching run of the step whose held strand is not live and has no pane: it is stopped and its strand removed.
`TestStopStrand_RelaunchingRecordStopsAndRemovesTheHeldStrand`: `StopStrand` on a relaunching record over a held, paneless strand records stopped and removes the strand.
`TestAttach_OutcomeDisposition`'s table gains a per-row strand status and a `RelaunchingAt` seed, which `seedAttachRunOpts` writes, and a row: a relaunching record within its bound over a held, paneless strand is attachable.
`TestReadAgentWaits` gains a row: a relaunching record within its bound with a dead pid reads live.
`TestStateCmd_ListsRunningRunsAndNarrowsByStrand` gains a row: a relaunching run's map carries `relaunching` true, and a running one's false.
`TestSweepOrphans_AgeGuardAndLiveGuid` gains two rows over a directory older than the age guard whose GUID is absent: a relaunching record within its bound is kept, and one past its bound is swept.

**ImpactSummary:** One tolerant field and one predicate reached by every running-record reader in the package, and `lym shuttle state` reporting it; two private helpers gain the clock and bound their two and three callers pass; the stop forms move to one locked read-modify-write.

**Commit:** 12: shuttle: a run record reads relaunching, through one running predicate
