# Card 13 — batten-drives-its-run

**Create:**
- `plan:internal/sheddriver#ReportFileName` -> `func ReportFileName(now time.Time, suffix string) string`
- `plan:internal/sheddriver#EnvelopeFileName` -> `func EnvelopeFileName(now time.Time, suffix string) string`
- `plan:internal/battencli#battenCLI.driveRun` -> `func (c *battenCLI) driveRun(cmd *cobra.Command) error`
- `plan:internal/battencli#battenCLI.driverDepsFor` -> `func (c *battenCLI) driverDepsFor(release func() error) (sheddriver.Deps, error)`
- `plan:internal/battencli#battenRunEnvelope` -> `func battenRunEnvelope(exit sheddriver.Exit, st shedengine.Status, found bool, abandonedSession, busyMessage string) (map[string]any, error)`
- `plan:internal/battencli#battenTacklerRecipe` -> `const battenTacklerRecipe = "batten"`
- `plan:internal/battencli#battenDriveRefusal` -> `const battenDriveRefusal = "battencli: a batten driver already drives this run"`
- `plan:internal/battencli#inProcessUntilStop` -> `func inProcessUntilStop(t *testing.T, c *battenCLI) func(ctx context.Context, attempt string) ([]byte, error)`
- `plan:internal/battencli#battenDriveLock` -> `func battenDriveLock(location *lyxcwd.Location, slug string) sheddriver.DriveLock`
- `plan:internal/battencli#TestBattenDriverDeps` -> `func TestBattenDriverDeps(t *testing.T)`

**Delete:**
- `internal/battencli#battenCLI.battenPostRun`
- `internal/battencli#TestBattenPostRun`

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#Arm`
- `internal/battencli#Command`
- `internal/battencli#battenVerbTexts`
- `internal/battencli#battenCLI.specFor`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#newFakeReceiver`
- `internal/battencli#TestRunCmd_ResumeDispositions`
- `internal/battencli#TestRunCmd_StateDoneRefusesNamingTheRunDir`
- `internal/battencli#TestRunCmd_HeldRunLockRefusesNamingTheLockPath`
- `internal/battencli#TestRunCmd_AbandonedSessionKey`
- `internal/loomcli#driverReportPath`
- `internal/loomcli#driveEnvelopePath`
- `pattern/PATTERN-shed-verb-set.md`

**Intent:**
`lyx batten run <slug>` runs `sheddriver`'s body in-process over its own run instead of the generic `run` body, so an unexpected batten stop meets the rule table and the tackler before the orch (D4).
Batten's window, pause, row notices, `abandonedSession` report and `haltedOverDoneChild` flag keep their behaviour.

The run verb:
- `Command` keeps `--window` as today and points the plain run at `driveRun`.
- `driveRun` first runs batten's existing pre-run (`battenPreRun`: the pause-coverage refusal, the done-slug refusal and the fresh-status seed), then `sheddriver.DriveVerb` over `battenDriveLock`, which names `shedrun.DriverLock(location, slug)`, the stale park marker and resume report, `battenDriveRefusal` (ending "; way forward: `lyx batten status <slug>` shows the run, and the live batten driver settles it") and the retry clause "re-run lyx batten run <slug>".
  The process holds the driver lock for its life, so a second `lyx batten run` for the slug is refused (D4).
- Inside the lock it runs `sheddriver.Run` over `driverDepsFor`, then builds the envelope with `battenRunEnvelope` from the `Exit`, the status as it then reads, and `readAbandonedSession` over the run's scratch directory.
- `battenRunEnvelope` keeps today's run envelope: `outcome` (the state of a done, blocked, paused or awaiting run), `halted_producer`, `reason` and `history_length`, plus `abandonedSession` when the teardown row recorded one.
  It returns an error carrying the status error for a failed run, and today's `RunBusyMessage` for `ExitBusy` and `ExitRunningElsewhere`.
  So a new-build run that meets an old-build batten process holding the run lock exits as busy, leaving that process to finish on its own build, and a rerun after it exited resumes at the recorded row (D4).
- `battenPostRun` and the receiver's `abandonedSession` field go, since the teardown now runs in a step child and the record file is the one source; `specFor` drops the `PostRun` hook, `wire`'s `Teardown.Shutdown` stops setting the field, and `Arm`'s doc stops naming it.
- `battenVerbTexts.Run.Long` says run drives the lifecycle through the Go driver: an unexpected stop meets the rule table and a cold tackler agent before it is handed to the orch, and a second run for the slug is refused while the first holds the driver lock.

`driverDepsFor` fills `sheddriver.Deps` for batten's run in the prime, every path from `shedrun` over the prime location and the slug (`PATTERN-shed-run-directory`).
It reads no config and builds no runner itself: the shed and batten configs, the model registry and the prime's shuttle runner load inside the `Tackler` closure on its first call, so a run that never reaches the tackler, every run test over a bare temp hub included, loads none of them.
- `Step` is `shedverbs.RunUntilStop` over the receiver's spec with argv `batten step <slug>` and the loop card 11 armed, unless the receiver's new `untilStop` override is set.
  A rerun after the process died while a detached step still runs adopts that step through the loop's waiter, since `runUntilStop` follows a live loop instead of spawning (D4).
- `Goto` is `shedverbs.RunGoto` over the spec, so `battenPreGoto` applies.
  The status, run-lock, pause and pause-hold reads are batten's own; the settle path is `shedrun.SettleRecords(location, slug)`, and the friction directory is empty.
  Saved envelopes and stop reports land in `shedrun.DriveReportsDir(location, slug)`, named by the new `sheddriver.ReportFileName` and `EnvelopeFileName`, which hold the `drive-report-<timestamp>-<4-hex>.md` and `drive-envelope-...json` patterns loom's `driverReportPath` and `driveEnvelopePath` now delegate to.
- `HandBack.Notify` sends through card 12's `battenshed.HandBackNotice` over `rowNotices(location, slug)`, with the stop's row and history length, and declines a paused stop, since a pause is the operator's own act (D4).
  `OrchRecorded` reads the prime's orch state (`orchStrandRecorded`).
  `Park` writes the park marker at `shedrun.ParkMarker(location, slug)` holding the report path and commits nothing.
- `Tackler` builds `sheddriver.NewTacklerCall` over `TacklerDeps`:
  - the tackler directory, attempt directories and marker from `shedrun`;
  - settings from `sheddriver.ResolveTackler` over the prime's `shed.yaml` and `batten.yaml`'s override (D5);
  - `Recipe: battenTacklerRecipe`, one line naming batten and what its run does, and `GotoRoutedFrom: false`, so a `resume` names the halted row only (D2);
  - no `Snapshot`;
  - `Shuttle` a `sheddriver.RunnerShuttle` over a `shuttleengine.NewRunner` built from the prime's own reed geometry and shuttle config, so the strand lives in the prime's reed session and Worktree-Teardown's session shutdown never ends it (D4).
    D4 calls it a detached runner; `shuttleengine.NewDetachedRunner` is the standalone shape whose disjointness check refuses hub geometry (`internal/shuttleengine/run.go`), so the prime's hub runner is the constructor that keeps the strand out of the task session.
  - the prompt's slug, prime worktree, trace directory and the batten process's own trace file as its log, with the stencils from `fabricengine.StencilsDir`.
- `ConsumeTackler` is `sheddriver.ConsumeTacklerAttempt` over batten's relaunch request; `OpenAttempts` uses `sheddriver.NewestTacklerAttempt` alone.
- `Arbiter`, `Verify`, `Watched`, `HandOver`, `Tamper`, `CommitRecords` and `WriteFrictionNote` stay nil (D2): with no arbiter an arbiter-bound stop and a tackler `escalate` hand back, the latter naming its case brief.
  A stop after the on-disk build changed and a blocked stop beside a Publish failure record reach the tackler, never a hand-over or a verify goto.
  No tamper snapshot runs, since the prime has other writers (D4); the stencil's write rule, which nothing enforces, is the only bound on what a batten tackler writes in the prime, the risk the record keeps open.

`PATTERN-shed-verb-set.md` gains D4's clause: a recipe's `run` verb may be `sheddriver`'s body, which steps through `shedverbs`' `--until-stop` and goto bodies and restates neither.

Tests:
- `newFakeReceiver` sets `untilStop` to `inProcessUntilStop`, which steps the receiver's shed in-process until it stops and renders the loop envelope the driver parses (`ok`, `state`, `reason`, and `loop` with `current_producer`, `history_length` and `state`), and the `kind: busy` error envelope for `shedengine.ErrShedBusy`.
  The four `TestRunCmd_*` tests keep their assertions over the driver body, and so pin `battenRunEnvelope`'s done, blocked and paused envelopes, `abandonedSession` present and absent, and the busy message.
  `TestRunCmd_ResumeDispositions` gains a failed-status row whose run ends in an error envelope carrying the status error, and `TestRunCmd_HeldRunLockRefusesNamingTheLockPath` gains a row where the in-process step reports the run running elsewhere, which also ends in the busy message.
  The abandoned-session test's fake shutdown no longer sets a receiver field, since the teardown row's record file carries the value.
- `TestBattenDriverDeps`, over a hand-populated receiver with a temp prime location:
  - nil `Arbiter`, `Verify`, `Watched`, `HandOver`, `Tamper`, `CommitRecords` and `WriteFrictionNote`;
  - `Tackler.Recipe` is `battenTacklerRecipe` and `GotoRoutedFrom` is false;
  - the settle, tackler, marker and report paths are `shedrun`'s for the prime and slug;
  - `battenDriveLock` names `shedrun.DriverLock` for the prime and slug, the park marker and resume report as stale files, and `battenDriveRefusal`;
  - `HandBack.Notify` declines a paused stop, reporting not sent with no error and writing no marker; the per-episode rule itself is card 12's `TestHandBackNotice`.
- The rule table, unwired-seam and hand-back rows of card 6 and the tackler rows of card 4 already pin the driver behaviour these seams select; the shedverbs loop tests pin the adoption of a live loop.
battencli's row in `cmd/lyx/testdata/test-budget.yaml` stays at 81: `TestBattenDriverDeps` replaces the deleted `TestBattenPostRun`.

**ImpactSummary:** batten's run verb body switches from the generic shed run to the sheddriver body with batten's seams; its post-run hook and receiver field go, two loom report-path composers delegate to sheddriver, and the run tests switch to an in-process step.
