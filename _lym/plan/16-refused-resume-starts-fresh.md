# Card 16 — A copied resume whose first turn ends on an API error starts fresh

**Create:**
- `plan:internal/shuttleengine#refusedResume` -> `func refusedResume(rs RunState, signal SessionSignal, firstTurnEnd, usageLimited bool) bool`
- `plan:internal/shuttleengine#Runner.RefusedResume` -> `func (r *Runner) RefusedResume(guid string, turnEnd Event) (bool, error)`
- `plan:internal/shuttleengine#TestWait_RefusedResumeStartsFresh` -> `func TestWait_RefusedResumeStartsFresh(t *testing.T)`
- `plan:internal/shuttleengine#loadRefusedResumeError` -> `type loadRefusedResumeError struct`

**Edit:**
- `internal/shuttleengine#Run.retryAPIError`
- `internal/shuttleengine#Run.readLoadTurn`
- `internal/shuttleengine#Run.loadSkillTurn`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`
- `internal/shuttleengine/doc.go`

**Extends:**
- `internal/shuttleengine#apiErrorWatch`

**Uses:**
- `plan:internal/shuttleengine#Run.relaunchUnderAccount`
- `internal/shuttleengine#UsageLimitReader`
- `internal/shuttleengine#FindRun`
- `plan:internal/shuttleengine#switchFakeEngine`
- `plan:internal/shuttleengine#TestWait_UsageLimitSwitchRelaunchesInTheRun`

**Intent:**
D6's refused resume: a resume whose check copied the transcript (`RunState.TranscriptCopied`, card 8) and whose first turn ends on an API error other than a usage limit is a refused resume, and starts the session fresh.
`refusedResume` holds when the record copied, the signal is an API-error turn end, it is the first turn end read past the record's `LaunchOffset` (card 8), a skill-load turn included, and the engine does not read it as a usage limit.
`apiErrorWatch` gains `turnEnds`, the turn ends read past `LaunchOffset`, so only the first qualifies, and the case runs ahead of `retryAPIError`'s `APIErrorRetries` return, so a spec without re-sends starts fresh too.
On the start path the case sits in `readLoadTurn`, since `loadSkillTurn`'s hold takes each API-error turn end there and `loadSkillsThenPrompt` sees only a passed load or the capped error.
The first load turn end past `LaunchOffset` that is an API error other than a usage limit, on a record with `TranscriptCopied`, ends the load hold uncounted: `loadSkillTurn` returns a `*loadRefusedResumeError`, and `loadSkillsThenPrompt` relaunches the run fresh through `relaunchUnderAccount(relaunchFresh, true)` instead of holding on a lasting API error.
It returns as card 14's start path returns: a relaunch that ends the run hands `start` an error wrapping `ErrNotStarted`, or, with every output file present, the unfinalized run for `Wait` to end done.
Like card 14's usage-limit case, the check runs ahead of both of `readLoadTurn`'s counting paths, the parser's `SessionSignalAPIErrorTurnEnd` read and the event loop's `TurnEndActivity` read, so the first API-error turn end is caught whichever path reads it.
In `retryAPIError`, such a turn end is neither counted nor re-sent: it runs `relaunchUnderAccount(relaunchFresh, false)` (card 14), a fresh launch in the same run, and a finished relaunch ends `Wait` with its result.
Any later API error, and an API error of a resume that copied nothing, are handled as today.
`internal/shuttleengine/doc.go` rewrites two paragraphs.
The skill-load hold paragraph, from "A load turn that ends on an API error" through "until a load turn ends", says a copied resume's first load turn end on an API error ends the hold and starts fresh.
The re-send paragraph that opens "A run whose Spec.APIErrorRetries is set re-sends such a turn" says such a first turn end is neither counted nor re-sent.
Its sentence "A run with no retries waits such a turn end out to its deadline" excepts the same first turn end, which starts fresh instead.
`loadSkillsThenPrompt`'s doc comment sentence "A load turn whose provider API fails skips nothing: loadSkillTurn holds it, and the prompt waits until a load turn ends" gains the copied resume's exception.
`Runner.RefusedResume(guid, turnEnd)` answers the same predicate for a run found by its strand GUID, reading its events file up to the turn end; the orch watcher asks it (card 17).

`TestWait_RefusedResumeStartsFresh` is tier0 on `switchFakeEngine`, built as `TestWait_UsageLimitSwitchRelaunchesInTheRun` builds its run:
- a copied resume whose first turn end is an API error relaunches fresh in the same run: a new session id in the record and in `RelaunchStrand`, the run's skills and prompt, and no API-error re-send recorded;
- the same with `TranscriptCopied` false re-sends as today;
- a copied resume whose first turn end is a usage limit is not refused;
- a copied resume whose second turn end is an API error re-sends as today;
- a copied resume on a spec without `APIErrorRetries` starts fresh;
- a copied resume whose skill-load turn ends on an API error starts fresh from the start path, with one load typed before the fresh relaunch;
- a `relaunchForReprompt` resume that copied, whose first turn past its `LaunchOffset` ends on an API error, starts fresh.
`Runner.RefusedResume` is covered by rows in the same test reading the same records through the GUID.

**ImpactSummary:** `retryAPIError` gains one case ahead of its counting; runs whose record carries no copy take no new path.

**Commit:** 16: shuttle: start a copied resume fresh when its first turn ends on an API error
