# Card 10 — seat-writer-resume-join

**Create:**
- `plan:internal/seatengine#writerResumeNotice` -> `func writerResumeNotice(name string, outputs []string) string`
- `plan:internal/seatengine#TestEngine_Resume_Writers` -> `func TestEngine_Resume_Writers(t *testing.T)`
- `plan:internal/seatengine#TestEngine_Run_WriterEndDecidesTheJoin` -> `func TestEngine_Run_WriterEndDecidesTheJoin(t *testing.T)`

**Edit:**
- `internal/seatengine#Engine.Resume`
- `internal/seatengine#advisorWork.outstanding`
- `internal/seatengine#Engine.join`
- `internal/seatengine#TestEngine_Run_ChairHoldDeferral`
- `internal/seatengine#TestEngine_Run_RequiredAdvisorEndsTheStep`
- `internal/seatengine/doc.go`

**Uses:**
- `plan:internal/seatengine#Seat.endsStep`
- `plan:internal/seatengine#Table.hasWriters`
- `internal/seatengine#fakeShuttle`
- `internal/seatengine#engineFixture`

**Intent:** Finish the writer seat's rules (D5, D6) in the engine's resume, hold deferral and join.

**Resume.** In `Engine.Resume`, a found writer is attached and watched as in a fresh run, with no up line queued.
A writer the probe did not find, which includes a done writer whose kept pane the probe removed, counts as finished when all its outputs exist and its Accept passes on a zero `shuttleengine.Result`; the chair is then told once with `writerResumeNotice`: "Writer <name> was stopped when this step was interrupted; any fix you had asked of it is lost, and it no longer answers; its outputs stand: <outputs>".
A writer not found whose outputs do not all exist, or whose Accept fails, ends the step at once as a required advisor not found does today, its Accept error kept on its result.
A required advisor not found keeps today's rule.

**Hold deferral.** `advisorWork.outstanding` counts a writer as it counts an advisor while its run has not ended, its up line aside, since a writer gets none: an unended writer is outstanding while its session reads mid-turn.
A writer whose run ended done still counts while its kept session reads mid-turn (busy for a turn), read the same way, so a chair waiting on a fix it asked of a done writer is deferred; any other ended run never counts.

**Join.** In `Engine.join`, for a table that `hasWriters`, a writer whose end ended the step is marked `EndedStep` even when the chair's end arrives first or together with it: in both cases of the join's select, once the chair's end is received and before any seat is stopped, a fired step ender marks its writer.
Stopping the kept writers after the chair's end never marks one, since the check runs before the stop.
For an advisor table `join` keeps today's rule that a chair that has also ended decides the step.

`TestEngine_Resume_Writers` (tier0, over `fakeShuttle`) covers what `TestEngine_Run_RequiredAdvisorEndsTheStep`'s not-found row does not: a live chair with a writer found keeps it and queues no line; a writer not found whose outputs exist and pass Accept queues the resume line and the step goes on until the chair ends.
`TestEngine_Run_RequiredAdvisorEndsTheStep` gains the writer variants of its resume row: a writer not found with an output missing, and one whose outputs exist but whose Accept fails, each end the step with `EndedStep` set and no line queued.
`TestEngine_Run_ChairHoldDeferral` gains rows: an unended writer reading mid-turn defers the chair's hold; a done writer reading mid-turn still defers it; a done writer reading idle does not.
`TestEngine_Run_WriterEndDecidesTheJoin` (tier0) calls `Engine.join` directly, as an internal test may, so no goroutine order decides its outcome: it builds the writer's run with `newSeatRun` over a handle released with a failed gate, waits until the step ender it passes has fired, releases the chair's handle with done, and only then calls `join`, so the chair's end and the ender are both ready whichever case the join's select takes.
The writer's result carries `EndedStep` in a writer table, and the same call over an advisor table with a required advisor in the writer's place leaves the outcome to the chair, today's rule for a chair that also ended.
`doc.go`'s `# Writers` and `# Resume` sections state these rules.

**ImpactSummary:** Resume, hold deferral and join gain writer-only branches; advisor tables keep every current rule.
