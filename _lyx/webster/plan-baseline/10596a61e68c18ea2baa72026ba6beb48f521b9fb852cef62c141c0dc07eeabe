# Card 4 — shuttle-session-usage

**Create:**
- `internal/shuttleengine#SessionUsage`
- `internal/shuttleengine#UsageReader`

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine/wait_test.go`
- `internal/shuttleengine/doc.go`

**Intent:**
The provider-neutral half of decision D8's token and wall-time reading (`PATTERN-shuttle-provider-seam`).
`SessionUsage`, in a new `internal/shuttleengine/sessionusage.go`, is one session's token reading with its forks included: `Known`, `Fresh` (input plus cache creation plus output tokens), `CacheRead`, and the fork part broken out as `Forks` (the fork transcript count), `ForkFresh` and `ForkCacheRead`.
`UsageReader` is an optional capability beside `Engine`, like `SessionCycler`: `SessionUsage(sessionID, workdir string) SessionUsage`, which never errors and returns an unknown reading when it cannot read.
`Result` gains `StartedAt` and `EndedAt` (`time.Time`) and `Usage SessionUsage`.
`Run.finalize` sets `StartedAt` from the run state's `CreatedAt` (left zero when it does not parse), `EndedAt` from the run's clock, and, on `OutcomeDone` only, `Usage` from the engine when it implements `UsageReader`, reading with the run's session ID and the pane cwd the fork audit uses.
An engine without the capability leaves `Usage` unknown; nothing about the outcome changes.

`wait_test.go` gains a test over a done run whose engine implements `UsageReader`, asserting the reading and both times reach the `Result`, and that a died run and an engine without the capability leave `Usage` unknown.
The `shuttleengine` package doc names the new optional capability beside the others.

**ImpactSummary:** Result gains three fields and finalize fills them; every existing Result consumer keeps compiling, and only engines that opt in report usage.
