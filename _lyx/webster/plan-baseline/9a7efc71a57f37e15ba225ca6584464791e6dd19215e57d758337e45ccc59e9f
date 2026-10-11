# Card 7 — shuttle's pending-send record keeps a retry from typing twice

**Create:**
- `plan:internal/shuttleengine#pendingSend` -> `type pendingSend struct`
- `plan:internal/shuttleengine#pendingSendFileName` -> `const pendingSendFileName = "pending-send.json"`
- `plan:internal/shuttleengine#savePendingSend` -> `func savePendingSend(runDir string, p pendingSend) error`
- `plan:internal/shuttleengine#loadPendingSend` -> `func loadPendingSend(runDir string) (pendingSend, bool, error)`
- `plan:internal/shuttleengine#clearPendingSend` -> `func clearPendingSend(runDir string) error`

**Edit:**
- `internal/shuttleengine#sendContext`
- `internal/shuttleengine#Run.newSendContext`
- `internal/shuttleengine#Runner.newSendContext` -> `func (r *Runner) newSendContext(state RunState, runDir string) sendContext`
- `internal/shuttleengine#Runner.Send`
- `internal/shuttleengine#Runner.LoadSkills`
- `internal/shuttleengine#Runner.SessionIdle`
- `internal/shuttleengine#Runner.TypeColor`
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#pasteVerified`
- `internal/shuttleengine#TestRun_Send_ConfirmsSubmission`
- `internal/shuttleengine/rundir_test.go`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#FindRun`
- `internal/shuttleengine#turnStartedSince`

**Intent:**
Record a send that may have left our text in the box, so its retry neither types it twice nor drops it (decision D5's pending-send record).

`pendingSend` holds `Text string` and `EventsOffset int64`, persisted as JSON in `pendingSendFileName` in the run's directory beside `run.json`, a file of its own so it never races `run.json`'s writers.
`savePendingSend` writes it atomically (temp file and rename, as `saveRunState` does); `loadPendingSend` answers found false for an absent file and an error for one that does not decode; `clearPendingSend` removes it and treats an absent file as done.

`sendContext` gains `runDir`.
`Run.newSendContext` fills it from the run; `Runner.newSendContext` takes it as a parameter, and its callers `Runner.Send`, `Runner.LoadSkills`, `Runner.SessionIdle` and `Runner.TypeColor` pass the run directory `FindRun` already returns.

`pasteVerified` saves `{text, sentAt}` when its result wraps `ErrSentTextInDraft` or `ErrSubmissionNotLanded` (both may leave our text in the box), and records nothing on a landing or on a pre-write hold, which typed nothing.
`sendVerified`, after `awaitIdleSession` passes on an empty box and before typing, loads the record for a paste-path engine.
A record whose `Text` equals the text being sent is cleared, and then the send returns landed without typing when `turnStartedSince(engine, eventsPath, EventsOffset)` holds (the text went out with the draft or with an Enter the operator pressed), and types afresh otherwise (the operator cleared it).
A record for another text is cleared and the send types as usual.
A record that cannot be read or cleared logs at Warn and the send types as usual, so a broken file never wedges a strand.
A slash command starts no turn, so its retry always types afresh (R8).

Bound: a record is read only by the next send to the same strand, which already passed the empty-box guard; an operator who deletes our text from the draft and sends the rest leaves a turn start that reads as ours, dropping that one retry (R6).

Tests:
- `TestRun_Send_ConfirmsSubmission` rows: a send failing with our text in the draft, retried after the box reads empty with a turn start past the recorded offset in the events file, types nothing and returns nil; the same retry with no turn start pastes once; a send failing not landed after a box-less window is retried the same way; a landed send leaves no record; a record for another text is cleared and the send pastes.
- `rundir_test.go`: save, load and clear round-trip, an absent file reads not found and clears without error, and a corrupt file is a load error.

Document the record in the verified-send section of `internal/shuttleengine/doc.go`.

**ImpactSummary:** One unexported constructor re-signed with its four callers in shuttleengine; a new file in each run directory.
