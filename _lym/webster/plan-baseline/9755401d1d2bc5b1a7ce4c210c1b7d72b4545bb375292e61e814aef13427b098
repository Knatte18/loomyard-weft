# Card 4 — sampler-session-row

**Create:**
- `internal/loomshed/samplersession_test.go`
- `plan:internal/loomshed#NameSamplerSession` -> `const NameSamplerSession = "Sampler-Session"`
- `plan:internal/loomshed#SamplerSessions` -> `type SamplerSessions interface`
- `plan:internal/loomshed#SamplerDeps` -> `type SamplerDeps struct`
- `plan:internal/loomshed#NewSamplerSession` -> `func NewSamplerSession(name string, deps SamplerDeps) shedengine.ShedProducer`
- `plan:internal/loomshed#fakeSamplerSessions` -> `type fakeSamplerSessions struct`
- `plan:internal/loomshed#TestSamplerSession` -> `func TestSamplerSession(t *testing.T)`
- `plan:internal/loomshed#samplerWorld` -> `type samplerWorld struct`

**Edit:**
- `internal/loomshed#loomshedAllowedImports`

**Extends:**
- `internal/loomshed/doc.go`

**Uses:**
- `internal/landingshed#PRReview.ensureSession`
- `internal/landingshed#PRReview.holdWhileCovered`
- `internal/pawlengine#BeamAwait.pause`
- `plan:internal/samplerengine#ReadRecord`
- `plan:internal/samplerengine#UpdateRecord`
- `internal/loomshed#NamePRReview`

**Intent:**
D5 and D6: the Sampler-Session row's producer, which keeps one interactive agent session in the run's worktree and waits in process, with no timeout, for the done flag.
It copies the shape of PR-Review's session code (`PRReview.ensureSession`, its stop and resume around reads) rather than sharing it, since the two rows keep different records.
`NameSamplerSession` joins loomshed's row-name constants.
`SamplerSessions` has `StartSession(spec shuttleengine.Spec) (shuttleengine.OpenResult, error)` and `StopStrand(guid string) error`, which `*shuttleengine.Runner` satisfies (asserted with `var _`).
`SamplerDeps` holds `RecordPath` and `RecordLockPath` (the sampler record), `Spec func() (shuttleengine.Spec, error)`, `Sessions SamplerSessions`, `SessionLive func(guid string) (bool, error)`, `RegisterSession func(guid string) error`, `Stamp func() (int, error)` (the run's current entry stamp of the row), `DirtyPaths func() ([]string, error)` (the code checkout's uncommitted paths), `CheckRecord func() ([]discussionparser.Finding, error)` (the decision record check Darn-Write's post-session guard runs), `MailAgent func(strandName, line string) error`, `PauseCovered func() (bool, error)`, `Now func() time.Time`, `Sleep func(ctx context.Context, d time.Duration) error` and `Watch <-chan struct{}`, nil leaving the wait on its timer.
`Call` returns an error naming the first unset seam but `Watch` (a fail-loud wiring guard).
Ensuring the session: a recorded strand that `SessionLive` reads live is reused; otherwise the spec is started with `ResumeSessionID` set to the recorded session id (empty for a first start) through `Sessions.StartSession`, the pane process is registered through `RegisterSession` (a failure logged, `PATTERN-run-process-owner`), and strand guid, name, session id and engine are re-recorded through `samplerengine.UpdateRecord`, so every start or resume of the row's own re-records the strand; a failed record write stops the new strand and is returned.
A start failure `shedtransient.Class` marks transient is an error the driver re-steps; any other is Stuck with Halt, reason naming the failure and the way forward: run `lym loom resume`; when the recorded session can no longer be resumed, commit the work in the task worktree and run `lym loom sampler-done` first, so the resume ends the row without a session, or end the run with `lym batten cancel <slug>`.
A `Spec()` error (an `agent` the registry cannot resolve, a missing stencil) is Stuck with Halt, reason naming the error and the way forward: fix the agent with `lym config sampler --set agent=<model-spec>` from the prime, or restore the stencil, then run `lym loom resume`.
`internal/loomshed`'s `loomshedAllowedImports` gains `internal/samplerengine` (told its record paths, resolves no geometry) and `internal/shedtransient` (classifies an error, resolves no geometry), each with that comment, so `TestToldGeometryInvariant_AllowlistOnly` passes.
The row's done judgement, reached when the record's `DoneStamp` equals `Stamp()` (an older stamp is ignored): first stop the recorded session when its strand is live, then read `DirtyPaths`; a dirty checkout clears the flag (`DoneStamp` 0), ensures the session (resuming it when the row stopped it), mails it through `MailAgent` a line naming the uncommitted paths and that `lym loom sampler-done` ends the build only on a committed worktree, and keeps waiting; a clean checkout with `CheckRecord` findings is Stuck with Halt, reason naming the findings and the existing `decisionRecordRestoreWayForward`; a clean checkout with an intact record is Done.
On entry, before it ensures a session, `Call` runs that judgement, so a flag written while halted at the row ends it Done on the resume without starting or resuming a session (also when the recorded session can no longer be resumed), and a session reopened while halted is stopped first.
With no current flag it ensures the session and waits.
Each wake of the wait: a wake `PauseCovered` reads covered stops the session when it is running and judges nothing else; the first uncovered wake after a covered one ensures the session before any other check, with no halt; a wake reading a current flag runs the done judgement; otherwise a recorded strand that is not live, when the row did not stop it, is Stuck with Halt naming `lym loom resume`, which re-enters the row and resumes the recorded session, and the same commit-then-`sampler-done` and `lym batten cancel <slug>` ends the start failure names, for a session that can no longer be resumed.
An unreadable pause record is logged and read as uncovered, as PR-Review does.
Wakes are at least a second apart, doubling to a minute while nothing changes; a `Watch` delivery cuts a sleep short and returns the interval to a second, so the verb's write takes effect within a second, in the shape of `BeamAwait.pause` (`PATTERN-daemon-wakeups`).
Every Done, Stuck or error the call ends with stops a still-live session, logging a failed stop, and a cancelled context is an error that leaves the session running, so a re-call after an interrupt finds it live.
`samplersession_test.go` carries `//go:build tier0`; `fakeSamplerSessions` records starts with their resume ids, stops and a live set, and the record lives in `t.TempDir()`.
`samplerWorld` holds the fake, the record paths and scripted seams: a per-wake `PauseCovered` and `DirtyPaths` script, a per-wake hook that may write the record (to raise a flag mid-wait) or kill a strand, a fake clock, and a `Sleep` that advances the clock by its interval and returns, cancelling the call's context after a row's wake budget, as `waitWorld.cancelAfter` does in `internal/landingshed/prreview_wait_test.go`.
A row whose wait never ends with a verdict (the dirty-flag row, the pause rows) runs to its wake budget and ends with the cancelled-context error; its assertions read the record, starts, stops and mails at that point.
The `Watch` row uses a `Sleep` that blocks until its context is cancelled, so only a `Watch` delivery can end the sleep, and asserts the next wake read the flag with the fake clock under a second past the delivery.
`TestSamplerSession` is a table, each row naming its seam scripts and asserting outcome, Halt, reason text, starts, stops, mails and the record:
first entry starts one session and records it (D5); a live recorded strand is reused with no start; a recorded session id whose strand is gone is resumed with that id and the new strand re-recorded; a non-transient start failure is Stuck with Halt naming `lym loom resume`, and a transient one is an error;
a flag at the current stamp written before the run halted ends the row Done on entry with no start, also when the session would fail to resume; a flag on entry with a live reopened strand stops it and ends Done; a flag at an older stamp is ignored and the session started; a flag read with a dirty checkout clears the flag, leaves the session live and mails it once; a flag with a damaged record is Stuck with Halt naming the restore steps, on entry with an unresumable session too;
a flag written during the wait (the fake writes it on the second wake) stops the session and ends Done; a strand that dies unstopped during the wait is Stuck with Halt naming `lym loom resume`; a covered wake stops the session and the next uncovered wake resumes it with no halt; a `Watch` delivery during a scripted long sleep ends the sleep and the next wake reads the flag, with the fake clock showing under a second of sleep; a cancelled context is an error with no stop;
the halt reasons of the start failure and the dead strand name `lym loom sampler-done` and `lym batten cancel`; a `Spec()` error is Stuck with Halt naming `lym config sampler`; a `RegisterSession` failure keeps the session and the row waiting; a failed record write after a start stops the new strand and is returned; an unreadable pause record reads as uncovered; a covered wake with a current flag stops the session and ends no verdict; a record read error ends the call with the error and the live session stopped.
`loomshed/doc.go` gains a paragraph on the Sampler-Session producer.

**ImpactSummary:** new producer and constant in loomshed with no caller yet; loomshed's import allowlist gains two geometry-free packages and its package doc a paragraph.

**Commit:** 4: loomshed: the Sampler-Session row's session and wait
