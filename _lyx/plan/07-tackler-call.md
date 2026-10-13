# Card 7 — tackler-call

**Create:**
- `internal/loomcli#tacklerDeps`
- `internal/loomcli#tacklerInput`
- `internal/loomcli#tacklerCall`
- `internal/loomcli#newTacklerCall`
- `internal/loomcli#tacklerCall.settle`
- `internal/loomcli#errTacklerUnavailable`
- `internal/loomcli#errTacklerResumeReturned`
- `internal/loomcli/tackler_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomcli#arbiterShuttle`
- `internal/loomcli#tacklerPrompt`
- `internal/loomcli#tacklerSpec`
- `internal/loomengine#TacklerAttempt`
- `internal/loomengine#TacklerEnvelope`
- `internal/loomengine#ListTacklerAttempts`
- `internal/loomengine#ReadTacklerVerdict`
- `internal/loomengine#TacklerMarker`
- `internal/loomengine#WorktreeSnapshot`

**Intent:**
D7: one cold tackler per stop, spawned through shuttle like the arbiter, its attempt records keyed by stop key so a restart never spawns a second agent for a stop already judged.
A new file `internal/loomcli/tackler.go` mirrors `arbiter.go`'s call shape; it moves no run itself.

`tacklerDeps` carries `Closed func(key loomengine.ArbiterStopKey, spawned time.Time) (bool, error)`, true when a `hand-back` settle record for the key is later than the attempt's spawn, and `ReadStatus`, `Settings loomengine.TacklerSettings`, `TacklerDir`, `AttemptDir func(int) string`, `MarkerPath`, `Producers`, `Snapshot func() (loomengine.WorktreeSnapshot, error)`, `PausedWithin func(from, to time.Time) (time.Duration, error)` (the time the hub's pause record covered the slug inside the window), the fixed prompt fields, `StencilsDir`, `Shuttle arbiterShuttle` and `Now`.
`tacklerInput` carries `RunID`, `EnvelopePath`, `ReportPath`, `TraceCopy` and `StderrPath`.

`newTacklerCall` reads the status, refusing a missing or unreadable one, and takes `loomengine.StopKeyOf` of it.
`settle` scans `ListTacklerAttempts` newest first for the stop key, skipping an attempt `Closed` reports closed: its stop was already handed to the parent, so a run the parent then resumed at the same key gets a fresh tackler (budget permitting) rather than the old verdict.
A closed attempt still counts toward `MaxSpawns`.
- An attempt for the key carrying the applied mark returns `errTacklerResumeReturned`: the driver's resume returned to the same stop, which D3 sends to the arbiter row.
- An attempt with a verdict on disk is concluded with no spawn.
- An attempt whose marker names it and whose deadline has passed is stopped through `Shuttle.StopStrand` (shuttle's stop verb, `PATTERN-shuttle-stop`) and concluded as `hand-back` with cause `timeout`.
- An attempt still within its deadline is waited on through `AttachIfLive` for the remaining time, and one with no live run and no verdict is concluded as `hand-back` with cause `missing`.
- With no attempt for the key, a zero `MaxSpawns` returns `errTacklerUnavailable` naming `tackler_max_spawns: 0`, and attempts on disk already at `MaxSpawns` return it naming the spent budget, so the driver hands back without a spawn.
- Otherwise it writes `attempt-<N>`'s record before the spawn: the stop key, the pause conditions, the worktree snapshot taken through `Snapshot`, the spawn time, the deadline, the envelope and report paths.
  It writes the prompt, logs `loom: tackler spawn` with the run, attempt, producer and deadline (`PATTERN-spawn-observability`), starts `tacklerSpec`, records the strand GUID and writes the in-flight marker (attempt, strand, deadline).
  A start error concludes as `hand-back` with cause `died`.
  A record or marker write failure stops the started strand first, as the arbiter's does.

The wait honors D11's paused time: when the wait ends in `OutcomeTimeout`, the call asks `PausedWithin(SpawnedAt, now)`; while the awake budget `now - SpawnedAt - paused` is still below `Settings.Timeout`, it moves the deadline out by the paused time, rewrites the attempt record and the marker, and attaches again for the remainder.
Otherwise it stops the strand through `StopStrand` and concludes `hand-back` with cause `timeout`.
A pause record it cannot read counts no paused time, with one Warn.

Concluding validates the verdict, and each of these reads as `hand-back` with the cause named:
- an unreadable or malformed verdict: `unparseable`;
- no verdict on disk: `missing`;
- an empty `question` on `hand-back`: `question`;
- an `escalate` whose case brief at `loomengine.TacklerCaseBriefPath` is missing or empty: `case-brief`;
- a `goto` naming a row other than the status's current row or the producer of the last history entry (the halting entry's routed-from row), or one `shedengine.CheckGotoTarget` refuses: `precondition`;
- a `resume` at a `blocked` stop with no `goto`: `precondition`.
A `hand-back` the call makes itself carries a question naming the cause, addressed to the run's parent.
Concluding removes the marker and writes the envelope into the attempt directory with the case brief path on an `escalate`.

Tampering is the spawn snapshot and a fresh one failing `WorktreeSnapshot.Equal`, which card 3 pins; the driver checks it only after its status re-read found the stop key unchanged (card 13).
Bound: the snapshot reads the existing tracked dirty-paths read, so a new untracked file is not seen as tampering.

Tests in `tackler_test.go`, over the `arbiterFake` shuttle from `arbiter_test.go`, a fake clock and a fake `PausedWithin`:
- `TestTacklerCall_SpawnPath`, a table: `MaxSpawns` 0 and a spent budget each return `errTacklerUnavailable` and start nothing; a fresh stop writes the attempt record before the start, the prompt and the marker, and the snapshot it recorded; a start error reads `hand-back`/`died`; a timeout with no paused time stops the strand and reads `timeout`; a timeout whose window a pause covered re-attaches with the deadline moved by the paused time, and the verdict written then is concluded.
- `TestTacklerCall_Rerun`, a table: a verdict on disk for the key is returned with no start; a live attempt within its deadline is attached; a marker past its deadline is stopped through `StopStrand` and reads `timeout`; an applied attempt returns `errTacklerResumeReturned`; an attempt with a verdict that `Closed` reports closed is skipped and a fresh attempt is spawned.
- `TestTacklerCall_VerdictOutcomes`, a table over verdict files: `resume` at an error stop, `resume` with a goto to the current row and to the last history entry's producer pass; a goto to another earlier row, and `resume` at `blocked` without a goto, read `precondition`; `hand-back` without a question reads `question`; `escalate` with an empty or absent brief reads `case-brief`, and with a brief carries its path; undecodable YAML reads `unparseable`.
`test-budget.yaml`'s `internal/loomcli` row rises by three.

**ImpactSummary:** a new unexported call in `internal/loomcli` with no caller until the Go driver's agents card; the budget row is the only existing file touched.
