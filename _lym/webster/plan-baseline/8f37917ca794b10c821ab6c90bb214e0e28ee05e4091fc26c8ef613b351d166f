# Card 5 — shuttle-unawaited-session

**Edit:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#Spec.validate`
- `internal/shuttleengine#TestSpec_Validate`
- `internal/shuttleengine#stopOutcome`
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine#dispositionCandidate`

**Extends:**
- `internal/shuttleengine/doc.go`

**Create:**
- `plan:internal/shuttleengine#Runner.StartSession` -> `func (r *Runner) StartSession(spec Spec) (OpenResult, error)`
- `plan:internal/shuttleengine#TestRunner_StartSession` -> `func TestRunner_StartSession(t *testing.T)`

**Uses:**
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#OpenResult`
- `internal/shuttleengine#Runner.StopStrand`
- `internal/shuttleengine#TestStartup_HandleReturned`

**Intent:**
D7: shuttle gains a start that carries the rendered prompt and is never awaited, and a resume of such a session by its session id; `Runner.Open` opens with no prompt and writes no run record, so it cannot serve.
`Spec` gains `Unawaited bool`: a session nobody waits on, with no output file, whose run record reads running until a stop ends it.
`Spec.validate` refuses `Unawaited` with a non-empty `OutputFiles` (`shuttle: spec.OutputFiles must be empty for an unawaited session, which has no output file`) and, for an unawaited spec only, waives the at-least-one-output-file rule; every other spec keeps it.
`Runner.StartSession` refuses a spec without `Unawaited` (`shuttle: StartSession needs Spec.Unawaited; use Start for a session whose output files are its result`), then runs the same `start` path `Start` runs, with no gate, so the session gets its run directory, run record, settings and shell guards, its skills and its startup gates, and returns once the provider is past startup, as an `OpenResult` carrying the strand GUID and name, the session id and the resolved engine; it never calls Wait.
A spec with `ResumeSessionID` set resumes that session through the engine's existing resume capability, which is the resume D7 names.
The returned strand is stopped through the existing `StopStrand`, which finds the run record and records the stop.
`stopOutcome` today keeps a record running when every output file exists, and an empty list counts as all existing, so an unawaited run would never read stopped; it changes so an empty `outputFiles` list, which only an unawaited run carries since `validate` refuses it for every other spec, gives `runOutcomeStopped` unless the outcome is done.
`wait.go`'s file header states that no `*Run` with an empty `OutputFiles` ever exists, its comments above `classifyDeadlineExpiry` and `finishedDespiteMechanismFailure` restate it, and so does the comment inside `attach.go`'s `dispositionCandidate`; each is rewritten: an empty `OutputFiles` exists only on an unawaited run, which never reaches Wait or Attach, since `StartSession` never calls Wait and Attach set-matches a persisted record's output files against the caller's spec, and every Attach caller passes an awaited spec naming its output files, which an unawaited record's empty list never matches.
An unawaited spec is ungated, so `start` hands reed its `OutputFiles` as `DoneWhen`, which is empty; reed drops a non-live strand only for a non-empty `DoneWhen` whose paths all exist, so a resume never drops an unawaited strand as finished and treats it like any other non-live strand.
Bound: the start is one session per call in the caller's worktree, with no timeout classification since nothing waits on it; the caller owns stopping it.
`TestSpec_Validate` gains rows: an unawaited spec with no output file passes; an unawaited spec with an output file is refused; an awaited spec with no output file is still refused.
`TestRunner_StartSession` uses the fakes `TestStartup_HandleReturned` runs on: a ready unawaited start returns the strand, session id and engine and writes a run record reading running; a spec without `Unawaited` is refused before any strand is added; a spec with `ResumeSessionID` passes that id to the engine's launch; `StopStrand` on the started unawaited run records it stopped and removes its strand, which would fail if the empty output list still kept it running.
The package doc gains a paragraph on the unawaited start, and its stop sentence, that a record still reading running whose output files all exist keeps running, gains that an empty list, an unawaited run's, reads stopped; `Spec.validate`'s doc comment, that `OutputFiles` must name at least one file, names the unawaited exception.

**ImpactSummary:** One new Runner method, one opt-in Spec field and one stop rule for an empty output list; every existing caller leaves the field false and names at least one output file, so validation, start and stop behave as today for them.

**Commit:** 5: shuttle: an unawaited session start with a prompt, resumable by session id
