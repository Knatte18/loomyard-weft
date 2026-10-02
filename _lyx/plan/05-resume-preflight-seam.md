# Card 5 — resume-preflight-seam

**Edit:**
- `internal/shuttleengine/engine.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/fakes_test.go`

**Create:**
- `internal/shuttleengine#SessionResumer`
- `internal/shuttleengine#Run.ResumeWarning`
- `internal/shuttleengine/run_resume_test.go`

**Intent:**
Add `SessionResumer` to `engine.go`, an optional capability beside `Engine` shaped like `SessionCycler`, with one method:
`CheckResume(sessionID, workdir string) (warning string, err error)`.
It answers whether the provider session `sessionID`, recorded under the pane cwd `workdir`, may be resumed by a new run:
an error refuses, and a non-empty warning means the check could not confirm something and proceeds.
Its doc names no provider specific, per the Shuttle Provider-Seam Invariant.

In `Runner.start`, after `spec.validate` and before the orphan sweep and `createRunDir`, a spec with a non-empty `ResumeSessionID` runs the check against `r.paneCwd`, the same told path the fork audit reads transcripts from.
An engine that does not implement `SessionResumer` is an error naming the missing capability, so no run can resume a session unchecked.
A refusal returns before any run directory or strand exists, wrapped as `shuttle: resume check: ...`.
A warning is logged through `logger.Warn` with the session id and kept on the returned `*Run`, read through `Run.ResumeWarning`.

Extend the fake engine in `fakes_test.go` with an opt-in resume check (recorded arguments, a canned warning and error) so the existing fake keeps satisfying only `Engine` unless a test asks for it.

Tests in `run_resume_test.go`: a resume spec calls the check with the session id and the runner's pane cwd before `Prepare`;
a refusal leaves no run directory and no `AddStrand` call;
a warning reaches `Run.ResumeWarning` and the run starts;
an engine without the capability refuses a resume spec;
a spec with no `ResumeSessionID` never calls the check.

**ImpactSummary:** One new optional interface and one guarded branch in Runner.start; specs without ResumeSessionID take the existing path untouched.
