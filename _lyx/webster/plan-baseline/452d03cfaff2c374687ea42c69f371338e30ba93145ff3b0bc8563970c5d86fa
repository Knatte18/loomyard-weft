# Card 12 — drive-loop

**Create:**
- `plan:internal/loomcli#goDriverDeps` -> `type goDriverDeps struct`
- `plan:internal/loomcli#goDriver` -> `type goDriver struct`
- `plan:internal/loomcli#goDriver.run` -> `func (d *goDriver) run(ctx context.Context) error`
- `plan:internal/loomcli#goDriver.facts` -> `func (d *goDriver) facts(stop driveStop) (driveFacts, error)`
- `plan:internal/loomcli#goDriver.recheck` -> `func (d *goDriver) recheck(stop driveStop) (next driveStop, exit bool, err error)`
- `plan:internal/loomcli#goDriver.holdPause` -> `func (d *goDriver) holdPause(ctx context.Context) error`
- `plan:internal/loomcli#goDriver.settle` -> `func (d *goDriver) settle(stop driveStop, settler, action, outcome, target string)`
- `plan:internal/loomcli#goDriver.handBack` -> `func (d *goDriver) handBack(stop driveStop, h driveHandBack) error`
- `internal/loomcli/drive_test.go`
- `plan:internal/loomcli#TestGoDriver_Settle` -> `func TestGoDriver_Settle(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/loomcli#decideDrive`
- `plan:internal/loomcli#parseDriveStop`
- `plan:internal/loomcli#stopFromStatus`
- `plan:internal/loomcli#handBackRun`
- `plan:internal/loomcli#driveNotices`
- `plan:internal/loomcli#readBattenWatch`
- `plan:internal/battenshed#RecordMatchesHalt`
- `plan:internal/loomengine#AppendSettleRecord`
- `plan:internal/loomengine#ReadSettleRecords`
- `plan:internal/loomengine#GoVerifyRetrySpent`

**Intent:**
The Go driver's settle loop for the stops Go settles by its own rule (D2, D3, D4, D9, D11, D12, D13), over injected seams so every branch runs at Tier 1.
A new file `internal/loomcli/drive.go` holds it; the tackler, the arbiter and the hand-over join it in cards 13 and 14, and until then those three actions hand back.

`goDriverDeps` carries, each a seam the verb wires in card 15:
- `RunID`;
- `Step(ctx, attempt string) ([]byte, error)`, running `shedverbs.RunUntilStop` into a buffer and returning the printed envelope;
- `Goto(ctx, target string) error`, wrapping `shedverbs.RunGoto`;
- `ReadStatus` and `RunLockHeld`;
- `SaveEnvelope(raw []byte) (string, error)`, which keeps each stop envelope under the run's drive-reports directory;
- `SettlePath`, the `shedrun.SettleRecordsFile` of the friction directory;
- `EscalationPending(producer string) (bool, error)`;
- `VerifyRecord`, `Head` and `VerifyRows`;
- `Watch() (battenWatch, bool)` and `BattenRetrySpent(w battenWatch) (bool, error)`;
- `Pause(now) (pauserecord.Coverage, error)` and `PauseHold(ctx) error`, the listed pause hold `pauserecord.WaitUncovered`;
- `NewReportPath() string` and `Park driveParkDeps`;
- `CommitRecords() error`;
- `WriteFrictionNote(stop driveStop) error`;
- `Now`.

`goDriver.run` loops:
1. `Step` with the attempt to consume, empty here, then `parseDriveStop` and `SaveEnvelope`.
2. `facts`: `EscalationPending` for an awaiting stop; for a `blocked` stop at one of `VerifyRows`, the record read, `Head` and `battenshed.RecordMatchesHalt` over the stop's status, the same test batten's publish retry applies.
   It reads `Watch`; `VerifyRetrySpent` is `loomengine.GoVerifyRetrySpent` over the settle records, or `BattenRetrySpent` when the watch names a prime, read whether or not that batten is alive.
   It also counts `RepairCount` from the settle records.
3. `decideDrive`.
4. Before carrying out any action but `done` and the busy exit, `recheck` re-reads the status (D12):
   - a `running` status, or a run lock another process holds, writes a `running-exit` settle record and exits;
   - a stop key that differs from the stop's is a moved run: a status whose last history entry is `shedengine.OutcomeGoto` is re-stepped, and any other moved status is decided afresh through `stopFromStatus`, each writing a `moved` record;
   - a moved status decided through `stopFromStatus` is saved through `SaveEnvelope` as a synthesized envelope carrying `state`, an empty `reason` and a `loop` object with `current_producer` and `history_length` from the status, and its path becomes the stop's `EnvelopePath`, so the tackler and the arbiter read an envelope for it;
   - a moved run is never parked and never gets a friction note.
5. Before a re-step, a goto, a spawn or a hand-over, `holdPause` reads `Pause`; while it covers the slug, it waits in `PauseHold` and then rechecks, so nothing moves under a pause (D11).
6. The action:
   - `done` commits the records through `CommitRecords` and exits; after the commit, a `friction: failed` done builds a `driveReport` (run-id, position, reason, the friction directory), takes `NewReportPath` and calls `handBackRun` with it, so the notice names a report on disk;
   - a busy exit writes a `busy-exit` record, logs the holder named in the envelope's error, and exits with no report and no park;
   - a re-step writes a `restep` record and loops;
   - the verify goto writes a `verify-retry` record naming the target, calls `Goto`, then loops to re-step; a refused goto is rechecked as a moved run;
   - every hand-back writes a `hand-back` record with settler `parent` and calls `handBack`, then exits.
     A hand-back carries `FrictionNote` through `WriteFrictionNote` first (D13).
`settle` appends one record (stop key, settler, action, outcome, time, target) and logs an append failure without stopping.
`handBack` builds the report from the stop, the decision's rule and the attempts so far, takes `NewReportPath`, and calls `handBackRun` with the watch state, so every hand-back leaves a report, a park marker and settle records (D9).

A `blocked` or `failed` halt keeps its Go-authored halt note and reflection from each `lyx loom step` child's `loomAfterStep`, which the loop's children run, so a go-driven halt still writes its note (`PATTERN-friction-capture`) without the driver writing a second one; `halt_test.go` already pins `loomAfterStep` under step.

Bound: the loop's waits are the `--until-stop` waiter, the pause hold and, from card 13, shuttle's `Wait`, all on `PATTERN-daemon-wakeups`' list.
It never retries a failed seam on its own; a failure it cannot act on hands back.

Test `TestGoDriver_Settle` in `drive_test.go`, a table over fake seams scripting the envelopes `Step` returns, the statuses `ReadStatus` returns and the settle file in a temp directory.
Every hand-back row asserts the report on disk, the park marker and a `hand-back` settle record with settler `parent`; which messages a hand-back sends is `TestDriveNotices`' (card 11), so the loop rows assert only that `Notify` was reached.
- `done` commits and exits with no park, and a `friction: failed` done writes its report on disk, reaches `Notify` and writes no park marker;
- `busy`, with the run lock held by another process, writes `busy-exit`, logs the holder and parks nothing;
- a step `paused` stop with an empty reason writes the friction note and parks, and one with a reason writes no note;
- `awaiting` with no notice parks;
- verify halts:
  - an unwatched verify halt with a matching record writes a `verify-retry` record, calls `Goto` with the record's target, re-steps, and a second such halt parks with no second goto;
  - a watched one parks with no goto;
  - one whose batten marker shows the retry spent parks with no goto;
  - one whose settle file already holds a `verify-retry` record from an earlier process parks with no goto;
  - a stale record from an earlier halt drives no goto;
- `interrupted` under `reinvoke` re-steps and writes `restep` records;
- moved runs:
  - a status re-read showing a goto entry re-steps with no note and no park;
  - one showing `running` exits with `running-exit`;
  - a run lock held by another process exits with `running-exit`;
- a pause covering the slug calls `PauseHold` before the re-step and steps nothing while covered, and one covering an unwatched verify halt calls `PauseHold` before `Goto`.
`test-budget.yaml`'s `internal/loomcli` row rises by one.

**ImpactSummary:** a new unexported loop in `internal/loomcli` with no caller until the `drive` verb of card 15; the budget row is the only existing file touched.
