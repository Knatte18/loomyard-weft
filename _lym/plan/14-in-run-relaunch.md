# Card 14 — Relaunch a switched session inside its run

**Create:**
- `plan:internal/shuttleengine#Run.stopForRelaunch` -> `func (run *Run) stopForRelaunch() (bool, error)`
- `plan:internal/shuttleengine#relaunchLaunch` -> `type relaunchLaunch int`
- `plan:internal/shuttleengine#relaunchContinue` -> `const relaunchContinue relaunchLaunch = iota`
- `plan:internal/shuttleengine#relaunchFresh` -> `const relaunchFresh`
- `plan:internal/shuttleengine#relaunchRepeat` -> `const relaunchRepeat`
- `plan:internal/shuttleengine#loadAccountSwitchedError` -> `type loadAccountSwitchedError struct`
- `plan:internal/shuttleengine#Run.relaunchUnderAccount` -> `func (run *Run) relaunchUnderAccount(launch relaunchLaunch, startPath bool) (Result, bool, error)`
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
- `internal/shuttleengine#Run.readLoadTurn`
- `internal/shuttleengine#Run.loadSkillTurn`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`
- `internal/shuttleengine#TestCompletionSignal_NegativeVerdictReturnSites`
- `internal/shuttleengine#TestCompletionSignal_FileContractCallSites`
- `pattern/PATTERN-shuttle-provider-seam.md`
- `PATTERN.md`
- `internal/shuttleengine/doc.go`

**Extends:**
- `internal/shuttleengine#apiErrorWatch`

**Uses:**
- `plan:internal/shuttleengine#AccountSwitcher`
- `plan:internal/shuttleengine#updateRunState`
- `plan:internal/shuttleengine#RunState.relaunching`
- `internal/shuttleengine#Runner.checkResume`
- `internal/shuttleengine#Run.sendStartPrompt`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#Run.identity`
- `internal/shuttleengine#Run.markedStopped`
- `internal/shuttleengine#removeStrandIfPresent`
- `internal/shuttleengine#Run.classifyDeadlineExpiry`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#resumeFakeEngine`
- `internal/shuttleengine#TestWait_APIErrorRetry`

**Intent:**
D7's in-run relaunch, inside `internal/shuttleengine` (`PATTERN-shuttle-provider-seam`).
In `Wait`'s held turn-end branch, ahead of the existing notice cases, a turn end `usageLimited` reads as a usage limit on an engine that implements `AccountSwitcher` goes to `switchAccountAtLimit`.
It asks the switch with the run directory and the turn end; on false, or on an error, which is logged at Warn, the turn end falls through to today's cases, and the limit is handled as today, `retryAPIError` counting and re-sending it.
On true it marks the turn end handled in `apiErrorWatch` (a new `handledOffset`, so `retryAPIError` neither counts it toward `APIErrorRetries` nor re-sends it), then runs `relaunchUnderAccount(relaunchContinue, false)`; a finished relaunch returns its result from `Wait`.

A usage limit also ends a skill-load turn, which `launch` runs through `loadSkillsThenPrompt` before `Wait`, and the load hold reads every API-error turn end there.
In `readLoadTurn`, a load turn end the engine's `TurnEndActivity` reads as a usage limit (`UsageLimit.Limited`), on an engine that implements `AccountSwitcher`, is asked of the switch with the run directory and the turn end before the load hold counts it, so every hit records `blocked_until` (card 13).
The case runs ahead of both of `readLoadTurn`'s counting paths: the parser's `SessionSignalAPIErrorTurnEnd` read, which counts through `apiErrorRead` before the event loop runs and which claude emits for a `StopFailure` hook with no usage-limit reading, and the event loop's `TurnEndActivity` read.
An API-error signal is therefore counted only after the session's newest turn end, read through `TurnEndActivity` as `Runner.UsageLimit` reads it, is found not to be a usage limit, also when no turn-end event follows the signal in the read.
On false, or on an error, which is logged at Warn, the load hold counts and retypes it as today.
On true the hold ends uncounted: `loadSkillTurn` returns a `*loadAccountSwitchedError`, and `loadSkillsThenPrompt` runs `relaunchUnderAccount(relaunchRepeat, true)`.
A relaunch whose pane starts returns `Result{}` and nil, so `launch` hands back the relaunched run.
A relaunch that ends the run returns as `abandonLoad` returns, so `start` never hands back a finished run as started and still judges its retry.
With `startPath` set, the relaunch finalizes none of its died or timeout ends itself, since `finalize` is not idempotent and `Wait` would finalize the returned run again.
When every output file exists (`classifyDeadlineExpiry`), it clears the mark, leaves the strand and returns `Result{}` and nil, so `Wait` ends the run done once.
Otherwise it ends through `abandonStartup` with the outcome and the failed step's error.
That finalizes once, removes the strand, and returns the Result beside an error wrapping `ErrNotStarted`.
A relaunch a stop ended removes the strand and returns as the record-write paragraph below states.
On the start path no caller holds the handle, so only `StopStrand` can end it;
it returns the run's `identity` with Outcome `OutcomeDied` beside an error wrapping `ErrNotStarted`.
It leaves `startupStrandRemoved` false, so `start`'s `startRetryable` never launches a stopped run again.

`relaunchUnderAccount` runs D7's steps in order:
1. `stopForRelaunch`, the stop verb's relaunch mode (in `stop.go`): in one `updateRunState` hold it writes `RelaunchingAt` unless the record is terminal (then it reports stopped and the relaunch ends as a stop-ended relaunch), then calls `reed.EndStrandProcess`, keeping the strand and its GUID.
   It records no outcome, so `Wait` keeps its run.
   When ending the process fails while the strand still reads live, it calls `reed.ReleaseStrandHold`, clears the mark back to running in one hold, and the limit is handled as today; when it fails over a strand no longer live, the relaunch goes on with step 2.
2. It builds the relaunch spec from the run's, by the launch it is given:
   - `relaunchContinue`: `ResumeSessionID` the record's session id, the run's resolved skills kept, `Prompt` the continue line `accountSwitchContinueLine`, `SkipSkillLoadTurn` set, so the launch's first turn is the continue line and its `PromptLine` is empty;
   - `relaunchFresh`: no `ResumeSessionID`, a new session id, the run's skills with their load turn, and the run's own prompt;
   - `relaunchRepeat`, for a limit on the skill-load turn by the parent's added decision: the run's own spec unchanged, with its skill-load turn and its deferred prompt line, so a fresh launch starts fresh again and a resume resumes the same session id, its transcript carried at step 3; no continue line is typed.

   `Prepare` writes the new account's `account.json` and status line (cards 6 and 10).
3. For a spec with `ResumeSessionID`, `checkResume` (card 8) carries the transcript and checks the resume; its live-holder test no longer sees the ended process; the copy fact is written into the record with step 4.
   A check refused with `ErrNoTranscript` runs steps 2 to 4 again as `relaunchFresh`.
   Any other refusal, or a failure in steps 2 to 4, ends the run as a strand that died: it finalizes `OutcomeDied` through the completion-signal check, so a run whose output files all exist ends done, and removes the strand.
4. In one `updateRunState` hold the record takes the launch's session id and `PromptLine`, `Started` clears, `PromptOffset` becomes the events file's size, `TranscriptCopied` the check's report, false for a fresh launch, and `LaunchOffset` (card 8) the same size as `PromptOffset`; then `reed.RelaunchStrand` replaces the stored `Cmd`, `ResumeCmd` and session id and relaunches the pane under the same GUID, so the record names the new launch before its pane exists.
   `awaitStartup(false)` decides on the pane capture alone, dismissing a trust gate as the start path does, since the kept events file already holds the limited session's session-start and turn-start lines; the start path and `relaunchForReprompt` pass true as today.
   A pane that never reaches ready ends the run as the start path's startup wait ends it: died, or timeout when the run's own deadline passes first.
   With `startPath` set, each died end of steps 2 to 4 and this timeout end go the start path's way the load-turn paragraph states instead.
   Once started, a non-empty `PromptLine` goes out as `launch` sends it (a fresh fallback loads its skills, then its prompt), and the mark clears back to running.
Every record write of the relaunch, the mark, step 4's write, the clear and the died end, goes through `updateRunState` and leaves a record a stop made terminal as it is; when a stop made it terminal, the relaunch removes the strand instead of writing.
It removes it through `removeStrandIfPresent`, also when the stop came before the mark, so a strand the stop failed to remove goes too;
a failed removal is logged at Warn.
A stop-ended relaunch then returns by the stop's form.
After the handle's `Run.Stop` (`markedStopped`), it finalizes `OutcomeDied`:
`finalize` keeps the record stopped through `stopOutcome`, stamps the run end and reports the run's one session end, and `Wait` returns that Result beside nil.
After `StopStrand`, which wrote the record and reported the session end itself, it finalizes nothing and returns the run's `identity` with Outcome `OutcomeDied`:
beside nil on the `Wait` path, and on the start path as the load-turn paragraph states.
When the relaunched strand starts, `apiErrorWatch` clears its count and pending turn end and keeps its cursor past the limited turn end, and `Wait` re-seeds its own cursors from the new `PromptOffset`.
The run ends once, with one session-end usage record.

`PATTERN-shuttle-provider-seam`'s Shuttle stop section gains the relaunch mode as the PATTERN change states: it marks the run record relaunching and keeps the strand; until the mark clears or its bound passes, every running-run reader reads it as live, a step stop stops it, and the orphan sweep keeps it.
Its done-when paragraph, which says reed's resume drops a non-live strand whose list is complete instead of relaunching it, gains the held strand's exception: reed's resume neither drops nor relaunches a strand held for a relaunch (card 11).
`PATTERN.md`'s entry line is unchanged:

```
- `PATTERN-shuttle-provider-seam` — Touching a provider or shuttle run: specifics only under `internal/shuttleengine/<engine>`, model-specs via `NewModelChoice`, a spec's prompt rendered for its launch's engine, a strand stopped via the stop verb, a negative finish after `allOutputFilesExist`. (test) — [background](pattern/PATTERN-shuttle-provider-seam.md)
```

`internal/shuttleengine/doc.go` documents the relaunch and rewrites three of its paragraphs.
The usage-limit paragraph that opens "A held turn end stopped by the provider's usage limit sends no notice" says an engine whose switch takes another account relaunches the run under it instead of holding.
The re-send paragraph that opens "A run whose Spec.APIErrorRetries is set re-sends such a turn" says a limited turn end the switch handles is neither counted nor re-sent.
Its sentence "A run with no retries waits such a turn end out to its deadline" excepts the same turn end, which relaunches instead.
The skill-load hold paragraph, from "A load turn that ends on an API error" through "until a load turn ends", says a usage-limited load turn end the switch handles ends the hold uncounted and repeats the launch.
`loadSkillsThenPrompt`'s doc comment sentence "A load turn whose provider API fails skips nothing: loadSkillTurn holds it, and the prompt waits until a load turn ends" gains the same exception and the start path's return for a relaunch that ends the run.

`TestWait_UsageLimitSwitchRelaunchesInTheRun` is tier0 on `fakeReed` and `switchFakeEngine`, a `resumeFakeEngine` that also implements `AccountSwitcher`, `UsageLimitReader` and `SessionSignalParser` with scripted answers and records `Prepare` specs, each row a scripted events file ending on a usage-limited held turn end:
- a switch relaunches inside the run: `EndStrandProcess` then `RelaunchStrand` with the second `Prepare`'s `Cmd`, `ResumeCmd` and the same session id; the record keeps its strand GUID, records no stopped outcome, holds an empty `PromptLine`, and the second spec resumes with the continue line as its prompt and no skill-load turn; no hold notice is sent;
- the relaunched pane shows a trust dialog over an events file already holding the old session-start and turn-start lines: the dismissal keys are played and the run is marked started only after a ready capture;
- a second `Prepare` that fails ends the run died and removes the strand;
- a relaunched pane that never reaches ready ends died, and timeout when the run's deadline is shorter;
- a check refused with `ErrNoTranscript` relaunches fresh: the record and `RelaunchStrand` name the new session id;
- a `StopStrand` run from `fakeReed`'s `OnEndStrandProcess` hook, between the mark and step 4, makes `Wait` return the run's identity with Outcome `OutcomeDied` beside nil, leaves the record stopped and no strand up, and reports one session end, the stop's;
- a `Run.Stop` on the handle from the same hook makes `Wait` return a finalized Result whose Outcome is `OutcomeDied` beside nil, leaves the record stopped and no strand up, and reports one session end, the run's;
- any check refusal other than `ErrNoTranscript` ends the run died and removes the strand;
- a record a `StopStrand` made terminal before the mark ends the relaunch with the run's identity and Outcome `OutcomeDied`, the record left stopped, no `EndStrandProcess` or `RelaunchStrand`, and no strand up;
- across a relaunch the run reports one session end;
- a usage-limited skill-load turn end driven through `Start` asks `SwitchAccount` with the run directory and that turn end before the load hold counts it, the call that records `blocked_until` (card 13); on true the run relaunches with a second `Prepare` spec equal to the first, the record holds the deferred `PromptLine`, and the load and the prompt are each typed once after the relaunch, with no load retyped before it; on false the load hold retypes the load as today;
- a usage-limited skill-load turn driven through `Start` with no start retries, whose relaunch's second `Prepare` fails, makes `Start` return no run and an error wrapping `ErrNotStarted`, the run finalized died once and its strand removed; the same with every output file present returns the run, whose `Wait` ends it done with one session end;
- a usage-limited skill-load turn driven through `start`, the seam `Start` calls, with start retries set, whose `fakeReed` `OnEndStrandProcess` hook runs `StopStrand`, returns no run and an error wrapping `ErrNotStarted`;
  the result is the run's identity with outcome died, the record reads stopped, no strand is up, one session end is reported, the stop's, and no second launch is made;
- an `EndStrandProcess` failing over a strand no longer live goes on with the relaunch;
- the tripwire tests pin `stopForRelaunch`'s and the relaunch's died and timeout ends as audited sites, each reached through the completion-signal check, the start path's `classifyDeadlineExpiry` and `abandonStartup` ends among them;
- a spec with `APIErrorRetries` records no re-send for the limited turn end and does not end `OutcomeAPIError`;
- a switch answering false sends no reed op and leaves the run as `TestWait_APIErrorRetry` pins it;
- an `EndStrandProcess` that fails releases the hold and clears the mark.

**ImpactSummary:** `Wait` gains one branch ahead of its notice cases and the load hold one case ahead of its counting; a relaunch from the start path that ends the run returns as a failed load does; `awaitStartup`'s two callers pass true as today; `shuttleengine/doc.go` rewrites three paragraphs; the stop verb gains a mode that keeps the strand; the PATTERN background's stop section gains one paragraph and its done-when paragraph one exception.

**Commit:** 14: shuttle: relaunch a usage-limited session under its switched account inside the run
