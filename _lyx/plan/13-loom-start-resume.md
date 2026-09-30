# Card 13 — `lyx loom start` resumes a parked driver

**Create:**
- `internal/loomcli/driverresume.go`
- `internal/loomcli/driverresume_test.go`

**Edit:**
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/run_test.go`
- `internal/loomcli/start.go`
- `internal/loomcli/driverlaunch.go`
- `internal/loomcli/cli.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/start_driver_test.go`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `internal/shedrun#ParkMarker`
- `internal/loomcli#driverResumeLine`
- `internal/loomcli#driverReportPath`
- `internal/loomcli#resolveDriverStrandAction`
- `internal/shuttleengine#Runner.Send`

**Intent:**
Make `lyx loom start` the resume route (decision record D8), so the hub's `lyx loom approve` then `lyx loom start` and batten's `Spawn` (`lyx loom start --no-attach`) keep their commands and never need to know a driver parked.

Readiness sentinel (`internal/shuttleengine/run.go`): a new exported `var ErrPaneNotReady = errors.New("shuttle: the strand's pane shows no input-ready provider")`, declared beside `requireReadyAgentPane`.
That function's not-ready refusal, the error naming "no input-ready provider TUI", wraps `ErrPaneNotReady` with `%w` and keeps its existing wording in the message, which the existing tests match by substring.
The capture-failure refusal and every other `Send` error do not wrap it.
This lets a caller wait on exactly the not-ready condition without matching error text.

Seams (`driverlaunch.go`, `cli.go`, `wiring.go`):
- A `driverSender` interface with `SendDriver(guid, text string) error`, implemented by `runnerDriverStarter` through `shuttleengine.Runner.Send`, which verifies delivery and refuses a pane whose provider is not input-ready.
- `loomCLI` gains a `driverSender` field and a `driverResumeWait func()` field (the pause between send attempts), both filled in `wiring.go` beside `driverStarter`.

`driverresume.go` holds `(*loomCLI).resumeParkedDriver(guid string) error`:
1. Compose a fresh report path with `driverReportPath` for the resolved run-id, and create its parent directory.
2. Build the line with `driverResumeLine`.
3. Call `SendDriver`, and while the error satisfies `errors.Is(err, shuttleengine.ErrPaneNotReady)`, run `driverResumeWait` and call it again, up to a declared attempt cap (`driverResumeSendAttempts`, about twenty).
   Production uses a declared interval of about three seconds, so the bounded wait is about a minute.
   Any other error ends the loop at once and is never retried: `Send` already replays its own keystrokes internally, and a "never appeared" verification failure can follow a delivery the pane hid, so re-sending could type the resume line into the driver twice.
   Every attempt is logged, and so is the outcome at the end, per Live-Substrate Spawn Observability; the loop caps the attempt count, not only elapsed time.
4. On a verified send, remove the marker at `shedrun.ParkMarker`, treating an already-absent marker as success (the driver may have removed it first), and log the resume with the report path.
5. On failure, return an error naming the attempts made, the last send error, and the retry (`run "lyx loom start" again once the driver's pane is ready`), leaving the marker in place.

`runDriverSpawnAndWait` (`start.go`) branches after it resolves the driver strand action, before any spawn:
- Spawn needed (`mustSpawnDriver` true, a dead or absent driver): remove a stale marker if present, then spawn exactly as today.
  A removal failure other than not-exist releases the bootstrap lock and refuses on the envelope.
- Driver strand live and the marker present: call `resumeParkedDriver`, spawn nothing.
  An error releases the bootstrap lock and refuses on the envelope; success returns true, so the handover runs as today (the envelope under `--no-attach`, otherwise the attach).
- Driver strand live and no marker: send nothing, spawn nothing, hand over as today.
The branch reads only strand liveness and the marker, never the seed's driver field a second time, per the Driver Choice Single-Site Invariant.

This branch carries the batten half of the decision record's end-to-end acceptance criterion.
A child parked at `awaiting` keeps its driver strand live, so after `lyx loom approve`, batten's Inner-Run row calls `Spawn`, which runs `lyx loom start --no-attach`.
That run takes the resume branch, the unchanged `--no-attach` handover (`noAttachFields`, pinned by `TestNoAttachFields_PinsSuccessEnvelope`) returns the success envelope once the send is verified, and `Spawn` succeeds with no second driver.
The `done` half is card 11's.

`startCmd`'s `Long` help (step 3 and the `--no-attach` paragraph) and the `--no-attach` flag text describe the three cases.
The help says `start` resumes a parked ly-drive driver by typing one line into its pane, and under `--no-attach` returns once that line's delivery is verified.
It also says `start` refuses after the bounded wait when the pane is not ready, leaving the driver parked.
`docs/overview.md`'s `start` description in the loom module entry and its "the bootstrap" entry say the same in one sentence each.

Tests (untagged, fakes only, no real pane and no real sleep):
- `run_test.go`: `TestRun_InterruptAndSend_RefuseAgentlessShellPane`'s `Send` case also asserts `errors.Is(err, ErrPaneNotReady)`, and a dead or untracked strand's refusal (`TestRun_InterruptAndSend_RefuseDeadOrUntrackedStrand`) asserts it does not.
- `driverresume_test.go`:
  - A fake sender that succeeds records exactly one call, whose text is a single line naming a path under the run's drive-reports directory, and the marker file is gone afterwards.
  - A sender failing twice with `ErrPaneNotReady` and then succeeding makes three calls with two waits.
  - A sender that always fails with `ErrPaneNotReady` makes exactly `driverResumeSendAttempts` calls; the error names a retry, and the marker file is still on disk.
  - A sender failing with any other error makes exactly one call and no wait, refuses, and leaves the marker on disk.
- `start_driver_test.go`, through `runDriverSpawnAndWait` with `newTestSpawnAndWaitReceiver` extended for the sender:
  - a live driver strand with the marker present sends once, removes the marker, never calls the starter, and returns true with no refusal recorded, which is the precondition for the `--no-attach` envelope batten's `Spawn` reads;
  - a live strand without the marker neither sends nor starts;
  - a dead strand with a stale marker removes the marker and starts a fresh driver;
  - a live strand with the marker and a sender that never succeeds refuses, releases the bootstrap lock, keeps the marker and never starts.

**ImpactSummary:** `lyx loom start` gains a resume branch taken only when a live driver strand and the park marker coexist, and removes a stale marker before a spawn; with no marker on disk, every existing path, including batten's `Spawn`, behaves exactly as before, and `shuttleengine`'s not-ready refusal keeps its wording while gaining a sentinel that no existing caller matches.

**Commit:** 13: loomcli: lyx loom start resumes a parked ly-drive driver
