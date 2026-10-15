# Card 6 — driver-loop-to-sheddriver

**Create:**
- `plan:internal/sheddriver#Deps` -> `type Deps struct`
- `plan:internal/sheddriver#Run` -> `func Run(ctx context.Context, deps Deps) (Exit, error)`
- `plan:internal/sheddriver#Exit` -> `type Exit string`
- `plan:internal/sheddriver#ExitDone` -> `const ExitDone Exit = "done"`
- `plan:internal/sheddriver#ExitHandBack` -> `const ExitHandBack Exit = "hand-back"`
- `plan:internal/sheddriver#ExitHandOver` -> `const ExitHandOver Exit = "hand-over"`
- `plan:internal/sheddriver#ExitBusy` -> `const ExitBusy Exit = "busy"`
- `plan:internal/sheddriver#ExitRunningElsewhere` -> `const ExitRunningElsewhere Exit = "running-elsewhere"`
- `plan:internal/sheddriver#DriveLock` -> `type DriveLock struct`
- `plan:internal/sheddriver#DriveVerb` -> `func DriveVerb(out io.Writer, lock DriveLock, run func(release func() error) (map[string]any, error)) int`
- `plan:internal/sheddriver#Stop` -> `type Stop struct`
- `plan:internal/sheddriver#Stop.Key` -> `func (s Stop) Key() StopKey`
- `plan:internal/sheddriver#HandBackDeps` -> `type HandBackDeps struct`
- `plan:internal/sheddriver#ArbiterSeam` -> `type ArbiterSeam struct`
- `plan:internal/sheddriver#ArbiterOutcome` -> `type ArbiterOutcome struct`
- `plan:internal/sheddriver#VerifySeam` -> `type VerifySeam struct`
- `plan:internal/sheddriver#HandOverSeam` -> `type HandOverSeam struct`
- `plan:internal/sheddriver#TamperSeam` -> `type TamperSeam struct`
- `plan:internal/sheddriver#NewestTacklerAttempt` -> `func NewestTacklerAttempt(tacklerDir string, key StopKey) (dir string, applied bool, err error)`
- `plan:internal/loomcli#loomPark` -> `func loomPark(commit commitStatusDeps, markerPath string) func(reportPath string) error`
- `plan:internal/loomcli#TestLoomPark` -> `func TestLoomPark(t *testing.T)`

**Move:**
- `internal/loomcli/drive.go`
- `internal/loomcli/drive_test.go`
- `internal/loomcli/driverules.go`
- `internal/loomcli/driverules_test.go`
- `internal/loomcli/drivehandback.go`
- `internal/loomcli/drivehandback_test.go`

**Edit:**
- `internal/loomcli/drivewiring.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Move the Go driver's settle loop, its closed rule table, the hand-back and the driver-lock verb out of `internal/loomcli` into `internal/sheddriver` (D1), and turn every loom-specific input into a seam the arming module fills (D2).
The rule table's rows and order, the repair cap of two re-steps per stop key, the verdict vocabulary, the settle and attempt records and the notify rules move unchanged; only their inputs become seams.

The three files and their tests move with `git mv` to the same names under `internal/sheddriver/`.
The boundary forces these edits, and no others:

- Exported names the arming modules use: `goDriverDeps` becomes `Deps`, `(*goDriver).run` becomes the entry `Run(ctx, deps)`, `driveStop` becomes `Stop` with `Key()`, `driveParkDeps` becomes `HandBackDeps`, and `driveVerb` becomes `DriveVerb`.
- `Run` also returns how the driver ended, an `Exit`: `ExitDone` at a done run, `ExitHandBack` after a hand-back, `ExitHandOver` after a hand-over, `ExitBusy` when the step reported another holder, and `ExitRunningElsewhere` when the recheck found the run running or its run lock held.
  The settle records and hand-backs are unchanged; the value only tells an arming module whose verb prints its own envelope (batten, card 13) a busy exit from a halt, and loom's `driveCmd` ignores it.
  The rule table (`decideDrive`, `driveFacts`, the actions, `binaryEligible`, `repairCount`), `parseDriveStop`, the report and the notices stay unexported.
- Log lines and errors say `sheddriver: drive:` for `loom: drive:`.
- `Deps` keeps today's required seams: run id, `Step`, `Goto`, `ReadStatus`, `RunLockHeld`, `SaveEnvelope`, `SettlePath`, `FrictionDir`, `NewReportPath`, `Pause`, `PauseHold`, `HandBack`, `Tackler`, `ConsumeTackler`, `OpenAttempts` and `Now`.
  `CommitRecords` and `WriteFrictionNote` become optional: nil commits nothing at a done stop and writes no `go-driver-paused` note.
- `HandBackDeps` holds `Notify func(stop Stop, line string) (sent bool, err error)`, `OrchRecorded`, `Park func(reportPath string) error` and `Now`.
  The hand-back composes the same notices as today; a notice the recipe declines (sent false, nil error) reads "not sent, the recipe sends none at this stop:" in the report's messages, beside today's sent, failed and not-queued lines.
  After the report is written, a hand-back that parks calls `Park`; nil parks nothing, and an error (the marker could not be written) is the hand-back's error, as the marker check is today.
- The optional groups, each nil-safe in the facts and the actions:
  - `Arbiter *ArbiterSeam` holds `EscalationPending` and `Judge func(envelopePath, reportPath, caseBriefPath string) (ArbiterOutcome, error)`, where `ArbiterOutcome` carries what the driver reads off loom's arbiter envelope (resume or not, reason, question, goto, cause, attempt, attempt directory, verdict path).
    With none, `EscalationPending` reads false, and an arbiter action hands back, with the rule "no arbiter is wired: hand back" and today's awaiting relay.
    A tackler `escalate` with none hands back with a question naming the case brief path, "The tackler escalated this stop and no stronger agent is wired; read its case brief at <path>, decide the stop yourself and report the choice in the stop report.", so the parent notice names the brief (D2).
  - `Verify *VerifySeam` holds `Halt func() (matches bool, target string, err error)`, read at a blocked stop, and an optional `RetrySpentElsewhere func() (bool, error)`, ORed with the settle records' `GoVerifyRetrySpent` as today's batten read is.
    With none, a verify halt never matches, so the verify-goto and watched-verify rows never fire.
  - `Watched func() bool`; nil is never watched.
  - `HandOver *HandOverSeam` holds `OnDiskBuild`, `BuildChanged`, `DirtyPaths`, `ReleaseLock`, `Executable` and `SpawnSuccessor`.
    With none the driver reads no build, `BuildChanged` and `HandedOver` stay false, and the hand-over row never fires.
  - `Tamper *TamperSeam` holds `Snapshot` and `AttemptSnapshot`.
    With none the tackler's verdict is acted on without the worktree comparison; `TacklerDeps.Snapshot` is likewise optional, nil recording an empty snapshot.
  These seams only remove rows or agents from the table: a stop a removed row would have caught meets the later rows unchanged, so it ends in the tackler under its spawn budget, a re-step under the repair cap, or a hand-back, and no unwired seam adds a resume path (D2's bound).
- `NewestTacklerAttempt` returns the newest attempt directory recorded for a key under a tackler directory and whether it carries the applied mark, the tackler half of loom's `newestAttemptsFor`, which now calls it; a recipe without an arbiter wires `OpenAttempts` from it alone.
- `DriveVerb` takes a `DriveLock` naming the lock path, the stale files, the busy refusal and the transient retry clause the recipe's messages end with; loom tells `driveRefusal` and "re-run lyx loom drive or lyx loom start", so loom's messages are byte-identical.
  Its `run` returns the success envelope's fields, so a recipe whose verb prints its own envelope (batten, card 13) reuses the lock handling; loom's `driveCmd` returns `{"drove": true}`, today's envelope.

In loomcli, `internal/loomcli/drivewiring.go` (which holds `driveCmd` since card 5, `goDriverDepsFor`, `writeDrivePausedNote` and `newestAttemptsFor`) is the one file that changes: `driveCmd` calls `sheddriver.DriveVerb` and `sheddriver.Run`, and `goDriverDepsFor` fills `sheddriver.Deps` with every group, keeping loom's behaviour.
`Arbiter` wraps `arbiterstop.EscalationEligible` and `newArbiterCall`, mapping `loomengine.ArbiterEnvelope` onto `ArbiterOutcome`.
`Verify` computes `battenshed.RecordMatchesHalt` over the Publish failure record, the HEAD and the status at `loomshed.NameMergeIn` and `loomshed.NamePublish`, and its `RetrySpentElsewhere` reads the batten-watched marker and `battenshed.PublishRetrySpent`.
`HandOver`, `Tamper` and `Watched` keep their closures.
`HandBack.Notify` sends through `orchcli.NotifyPrime`, ignoring the stop.
`HandBack.Park` is the new `loomPark` in `drivewiring.go`, which runs `commitAndPark` and returns an error only when the park marker does not hold the report path afterwards, today's check in `handBackRun`; `writeDrivePausedNote` takes a `sheddriver.Stop`.
`TestLoomPark`, in a new `internal/loomcli/drivewiring_test.go`, keeps the coverage `TestDriveHandBack` loses when the commit leaves for the seam, over a fake `commitStatusDeps`: a clean commit writes the marker and returns nil; a failing commit still writes the marker and returns nil; a marker path whose directory cannot be created returns an error naming it.

The moved tests adapt to the seams: `TestGoDriver_Settle`'s harness fills the optional groups, its verify rows script `Verify.Halt` with a plain row name instead of a `battenshed` record and `loomshed` constant, and `TestDriveHandBack` scripts `Park`.
The harness's `Park` writes the report path into its marker file, as `commitAndPark` did, so every `wantParked` row keeps its assertion, and its `Snapshot` and `AttemptSnapshot` fakes append `snapshot` to the recorded calls.
Its table gains `noArbiter`, `noVerify`, `noHandOver`, `noTamper` and `noCommit` fields, each leaving its group or seam nil for the row.
The report renders an empty `FrictionDir` as `none`, as the tackler prompt renders an absent path; a hand-back row with `FrictionDir` empty asserts "Friction directory: none".
`TestGoDriver_Settle` gains rows for the unwired seams, each asserting its calls, settle actions and report:
- no arbiter: a returned tackler resume (`OpenAttempts` reporting applied) hands back with no arbiter call;
- no arbiter: a tackler `escalate` hands back and the stop report's question names the case brief path;
- no hand-over seam: a blocked stop whose last step ran another build goes to the tackler, with no successor spawned;
- no verify seam: a blocked stop goes to the tackler, never a goto;
- no tamper seam: a tackler resume is consumed, its calls holding no `snapshot` and its attempt note no `tampered` cause;
- no commit seam: a done run exits with no commit.
`TestDriveHandBack` gains the declined-notice row.
`TestGoDriver_Settle` asserts each row's `Exit` beside its calls, through a new `wantExit` field, so a busy row reads `ExitBusy` and a running-elsewhere row `ExitRunningElsewhere`.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/sheddriver` row rises from 9 to 15 for the six moved tests; loomcli's row covers `TestLoomPark`, since the moves take twelve of its tests.

**ImpactSummary:** the drive loop, rule table and hand-back move to sheddriver behind optional seams; loom's drive verb and wiring are the only callers and keep loom's behaviour, with one new loomcli helper for the commit-and-park.
