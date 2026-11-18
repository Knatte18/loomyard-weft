# Card 16 — A copied resume whose first turn ends on an API error starts fresh

**Create:**
- `plan:internal/shuttleengine#refusedResume` -> `func refusedResume(rs RunState, signal SessionSignal, firstTurnEnd, usageLimited bool) bool`
- `plan:internal/shuttleengine#Runner.RefusedResume` -> `func (r *Runner) RefusedResume(guid string, turnEnd Event) (bool, error)`
- `plan:internal/shuttleengine#TestWait_RefusedResumeStartsFresh` -> `func TestWait_RefusedResumeStartsFresh(t *testing.T)`

**Edit:**
- `internal/shuttleengine#Run.retryAPIError`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`

**Extends:**
- `internal/shuttleengine#apiErrorWatch`
- `internal/shuttleengine/doc.go`

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
On the start path a copied resume whose skill-load turn ends on such an API error is refused the same way: `loadSkillsThenPrompt` relaunches it fresh inside the run through `relaunchUnderAccount(true)` instead of holding on a lasting API error.
In `retryAPIError`, such a turn end is neither counted nor re-sent: it runs `relaunchUnderAccount(true)` (card 14), a fresh launch in the same run, and a finished relaunch ends `Wait` with its result.
Any later API error, and an API error of a resume that copied nothing, are handled as today.
`Runner.RefusedResume(guid, turnEnd)` answers the same predicate for a run found by its strand GUID, reading its events file up to the turn end; the orch watcher asks it (card 17).

`TestWait_RefusedResumeStartsFresh` is tier0 on `switchFakeEngine`, built as `TestWait_UsageLimitSwitchRelaunchesInTheRun` builds its run:
- a copied resume whose first turn end is an API error relaunches fresh in the same run: a new session id in the record and in `RelaunchStrand`, the run's skills and prompt, and no API-error re-send recorded;
- the same with `TranscriptCopied` false re-sends as today;
- a copied resume whose first turn end is a usage limit is not refused;
- a copied resume whose second turn end is an API error re-sends as today;
- a copied resume on a spec without `APIErrorRetries` starts fresh;
- a copied resume whose skill-load turn ends on an API error starts fresh from the start path;
- a `relaunchForReprompt` resume that copied, whose first turn past its `LaunchOffset` ends on an API error, starts fresh.
`Runner.RefusedResume` is covered by rows in the same test reading the same records through the GUID.

**ImpactSummary:** `retryAPIError` gains one case ahead of its counting; runs whose record carries no copy take no new path.

**Commit:** 16: shuttle: start a copied resume fresh when its first turn ends on an API error
