# Card 10 — a run start and the deferred prompt wait out a pause that refused their line

**Create:**
- `plan:internal/shuttleengine#Run.sendThroughPause` -> `func (run *Run) sendThroughPause(sc sendContext, text string) error`

**Edit:**
- `internal/shuttleengine#Runner.waitWhileCovered` -> `func (r *Runner) waitWhileCovered(slug string) error`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`
- `internal/shuttleengine#Run.loadSkillTurn`
- `internal/shuttleengine#Run.DeliverPendingPrompt`
- `internal/shuttleengine#TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted`
- `internal/shuttleengine#TestRunner_Start_WaitsWhileAPauseCoversItsPair`

**Uses:**
- `internal/shuttleengine#PausedError`
- `internal/shuttleengine#newFixture`
- `internal/shuttleengine#readyStart`
- `internal/shuttleengine#startSendEngine`
- `internal/shuttleengine#startPasteEngine`
- `internal/shuttleengine#coverScope`
- `internal/shuttleengine#liftClock`
- `internal/shuttleengine#newFakeClock`

**Intent:**
D2 for the run start's prompt line, its typed skill-load line, and the deferred task prompt.
`waitWhileCovered` takes the slug it waits on: `Runner.start` passes the runner's geometry slug, as today.
`sendThroughPause` sends `text` through `sendVerified` over `sc`, and on a `*PausedError` waits through `waitWhileCovered` on the error's `Slug`, the slug the refusal judged, then sends again with a fresh context of the same kind, so the wait and the refusal judge the same slug; any other result returns as `sendVerified` gave it, and the wait's own error returns wrapped.
It loops, so a second pause landing between the wait and the resend holds again.
`loadSkillsThenPrompt` delivers its prompt line, and `loadSkillTurn` its typed skill-load line, through `sendThroughPause`, so a pause landing after the start's first wait holds the start instead of tearing it down.
The segment-color line `typeStartupColor` types, whether `Runner.start` or `loadSkillsThenPrompt` calls it, is not held: a refusal with `ErrPaused` is logged and dropped as any other color failure is, since the color is cosmetic and starts no turn.
`DeliverPendingPrompt`, which burler's `ResumeFix` reaches after the run start's own wait, sends its line through `sendThroughPause` under its `sendMu` hold, so `ResumeFix` never receives the refusal.
The orch runner, told no geometry, never gets `ErrPaused`, so `lyx orch start` keeps typing its start lines under a hub entry.
The run's deadline already excludes the paused interval through `observePause`.

`TestRunner_Start_WaitsWhileAPauseCoversItsPair` already drives a start under a pause with the same `coverHub`, `liftClock` and `readyStart` setup; it becomes a table whose first row is its current scenario, the start's own wait, and gains rows for a pause that lands after that wait.
In those rows the pane reads busy for the prompt send's first idle polls (the paste engine's busy reading), and a `liftClock` sleep among those polls adds the entry, since a pane that reads idle at once leaves no sleep between the start's wait and the send; a later sleep resumes it:
a hub entry added during the prompt's idle wait types the prompt only after the resume, and Start returns the run;
a slug entry of `p1` over a strand named `ly:p1:impl`, from a runner whose geometry slug is `other`, holds the prompt until that `p1` entry resumes and types it once;
a second entry added during the first wait-and-resend, after the first entry resumes and before the resend, holds the prompt again until it resumes too, and the prompt is typed once;
a spec with a skill whose load turn reports it missing holds the typed skill-load line the same way.
`TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted` gains a row under a covering hub entry resumed at a later clock sleep: the prompt is typed once, after the resume, and the call reports true.

**ImpactSummary:** one private wait re-signed with its one caller; three send sites in the start and resume paths go through one new helper; no exported surface changes.
