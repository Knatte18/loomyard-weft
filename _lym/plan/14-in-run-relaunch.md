# Card 14 — Relaunch a switched session inside its run

**Create:**
- `plan:internal/shuttleengine#Run.stopForRelaunch` -> `func (run *Run) stopForRelaunch() (bool, error)`
- `plan:internal/shuttleengine#Run.relaunchUnderAccount` -> `func (run *Run) relaunchUnderAccount(fresh bool) (Result, bool, error)`
- `plan:internal/shuttleengine#Run.switchAccountAtLimit` -> `func (run *Run) switchAccountAtLimit(held *heldTurnEnd) (Result, bool, bool, error)`
- `plan:internal/shuttleengine#accountSwitchContinueLine` -> `const accountSwitchContinueLine = "[lym] Your Claude account was switched after a usage limit. Carry on with your task where you stopped."`
- `plan:internal/shuttleengine#switchFakeEngine` -> `type switchFakeEngine struct`
- `plan:internal/shuttleengine#TestWait_UsageLimitSwitchRelaunchesInTheRun` -> `func TestWait_UsageLimitSwitchRelaunchesInTheRun(t *testing.T)`

**Edit:**
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.retryAPIError`
- `internal/shuttleengine#Run.awaitStartup` -> `func (run *Run) awaitStartup(eventsProof bool) (Result, error)`
- `internal/shuttleengine#Runner.launch`
- `internal/shuttleengine#Run.relaunchForReprompt`
- `internal/shuttleengine#TestCompletionSignal_NegativeVerdictReturnSites`
- `internal/shuttleengine#TestCompletionSignal_FileContractCallSites`
- `pattern/PATTERN-shuttle-provider-seam.md`
- `PATTERN.md`

**Extends:**
- `internal/shuttleengine#apiErrorWatch`
- `internal/shuttleengine/doc.go`

**Uses:**
- `plan:internal/shuttleengine#AccountSwitcher`
- `plan:internal/shuttleengine#updateRunState`
- `plan:internal/shuttleengine#RunState.relaunching`
- `internal/shuttleengine#Runner.checkResume`
- `internal/shuttleengine#Run.sendStartPrompt`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#resumeFakeEngine`
- `internal/shuttleengine#TestWait_APIErrorRetry`

**Intent:**
D7's in-run relaunch, inside `internal/shuttleengine` (`PATTERN-shuttle-provider-seam`).
In `Wait`'s held turn-end branch, ahead of the existing notice cases, a turn end `usageLimited` reads as a usage limit on an engine that implements `AccountSwitcher` goes to `switchAccountAtLimit`.
It asks the switch with the run directory and the turn end; on false, or on an error, which is logged at Warn, the turn end falls through to today's cases, and the limit is handled as today, `retryAPIError` counting and re-sending it.
On true it marks the turn end handled in `apiErrorWatch` (a new `handledOffset`, so `retryAPIError` neither counts it toward `APIErrorRetries` nor re-sends it), then runs `relaunchUnderAccount(false)`; a finished relaunch returns its result from `Wait`.

`relaunchUnderAccount` runs D7's steps in order:
1. `stopForRelaunch`, the stop verb's relaunch mode (in `stop.go`): in one `updateRunState` hold it writes `RelaunchingAt` unless the record is terminal (then it reports stopped and the relaunch ends with the stop's outcome), then calls `reed.EndStrandProcess`, keeping the strand and its GUID.
   It records no outcome, so `Wait` keeps its run.
   When ending the process fails while the strand still reads live, it calls `reed.ReleaseStrandHold`, clears the mark back to running in one hold, and the limit is handled as today; when it fails over a strand no longer live, the relaunch goes on with step 2.
2. It builds the relaunch spec from the run's: `ResumeSessionID` the record's session id, the run's resolved skills kept, `Prompt` the continue line `accountSwitchContinueLine`, `SkipSkillLoadTurn` set, so the launch's first turn is the continue line and its `PromptLine` is empty; `Prepare` writes the new account's `account.json` and status line (cards 6 and 10).
   When the record holds a deferred `PromptLine` whose own turn never started, as when the skill-load turn ended on the limit, the relaunch keeps that line in the record at step 4 and types it once started, in place of the continue line, so the task is never lost.
3. `checkResume` (card 8) carries the transcript and checks the resume; its live-holder test no longer sees the ended process; the copy fact is written into the record with step 4.
   A check refused with `ErrNoTranscript` runs steps 2 to 4 again with `fresh` set: no `ResumeSessionID`, a new session id, the run's skills with their load turn, and the run's own prompt.
   Any other refusal, or a failure in steps 2 to 4, ends the run as a strand that died: it finalizes `OutcomeDied` through the completion-signal check, so a run whose output files all exist ends done, and removes the strand.
4. In one `updateRunState` hold the record takes the launch's session id and `PromptLine`, `Started` clears, `PromptOffset` becomes the events file's size, `TranscriptCopied` the check's report, false for a fresh launch, and `LaunchOffset` (card 8) the same size as `PromptOffset`; then `reed.RelaunchStrand` replaces the stored `Cmd`, `ResumeCmd` and session id and relaunches the pane under the same GUID, so the record names the new launch before its pane exists.
   `awaitStartup(false)` decides on the pane capture alone, dismissing a trust gate as the start path does, since the kept events file already holds the limited session's session-start and turn-start lines; the start path and `relaunchForReprompt` pass true as today.
   A pane that never reaches ready ends the run as the start path's startup wait ends it: died, or timeout when the run's own deadline passes first.
   Once started, a non-empty `PromptLine` goes out as `launch` sends it (a fresh fallback loads its skills, then its prompt), and the mark clears back to running.
Every record write of the relaunch, the mark, step 4's write, the clear and the died end, goes through `updateRunState` and leaves a record a stop made terminal as it is; when a stop made it terminal, the relaunch removes the strand instead of writing.
When the relaunched strand starts, `apiErrorWatch` clears its count and pending turn end and keeps its cursor past the limited turn end, and `Wait` re-seeds its own cursors from the new `PromptOffset`.
The run ends once, with one session-end usage record.

`PATTERN-shuttle-provider-seam`'s Shuttle stop section gains the relaunch mode as the PATTERN change states: it marks the run record relaunching and keeps the strand; until the mark clears or its bound passes, every running-run reader reads it as live, a step stop stops it, and the orphan sweep keeps it.
`PATTERN.md`'s entry line is unchanged:

```
- `PATTERN-shuttle-provider-seam` — Touching a provider or shuttle run: specifics only under `internal/shuttleengine/<engine>`, model-specs via `NewModelChoice`, a spec's prompt rendered for its launch's engine, a strand stopped via the stop verb, a negative finish after `allOutputFilesExist`. (test) — [background](pattern/PATTERN-shuttle-provider-seam.md)
```

`TestWait_UsageLimitSwitchRelaunchesInTheRun` is tier0 on `fakeReed` and `switchFakeEngine`, a `resumeFakeEngine` that also implements `AccountSwitcher`, `UsageLimitReader` and `SessionSignalParser` with scripted answers and records `Prepare` specs, each row a scripted events file ending on a usage-limited held turn end:
- a switch relaunches inside the run: `EndStrandProcess` then `RelaunchStrand` with the second `Prepare`'s `Cmd`, `ResumeCmd` and the same session id; the record keeps its strand GUID, records no stopped outcome, holds an empty `PromptLine`, and the second spec resumes with the continue line as its prompt and no skill-load turn; no hold notice is sent;
- the relaunched pane shows a trust dialog over an events file already holding the old session-start and turn-start lines: the dismissal keys are played and the run is marked started only after a ready capture;
- a second `Prepare` that fails ends the run died and removes the strand;
- a relaunched pane that never reaches ready ends died, and timeout when the run's deadline is shorter;
- a check refused with `ErrNoTranscript` relaunches fresh: the record and `RelaunchStrand` name the new session id;
- a `StopStrand` run from `fakeReed`'s `OnEndStrandProcess` hook, between the mark and step 4, leaves the record stopped and no strand up;
- any check refusal other than `ErrNoTranscript` ends the run died and removes the strand;
- a record a stop made terminal before the mark ends the relaunch with the stop's outcome and no reed op;
- across a relaunch the run reports one session end;
- a limit hit during the skill-load turn keeps the deferred prompt line and types it after the relaunch;
- an `EndStrandProcess` failing over a strand no longer live goes on with the relaunch;
- the tripwire tests pin `stopForRelaunch`'s and the relaunch's died and timeout ends as audited sites, each reached through the completion-signal check;
- a spec with `APIErrorRetries` records no re-send for the limited turn end and does not end `OutcomeAPIError`;
- a switch answering false sends no reed op and leaves the run as `TestWait_APIErrorRetry` pins it;
- an `EndStrandProcess` that fails releases the hold and clears the mark.

**ImpactSummary:** `Wait` gains one branch ahead of its notice cases; `awaitStartup`'s two callers pass true as today; the stop verb gains a mode that keeps the strand; the PATTERN background's stop section gains one paragraph.

**Commit:** 14: shuttle: relaunch a usage-limited session under its switched account inside the run
