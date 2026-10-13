# Card 10 — drive-rules

**Create:**
- `internal/loomcli#driveStop`
- `internal/loomcli#parseDriveStop`
- `internal/loomcli#stopFromStatus`
- `internal/loomcli#driveStop.key`
- `internal/loomcli#driveFacts`
- `internal/loomcli#driveAction`
- `internal/loomcli#driveDecision`
- `internal/loomcli#decideDrive`
- `internal/loomcli#binaryEligible`
- `internal/loomcli#repairCount`
- `internal/loomcli#driveRepairCap`
- `internal/loomcli/driverules_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomengine#SettleRecord`
- `internal/loomengine#SettleRestep`
- `internal/loomengine#StopKeyOf`

**Intent:**
D3's closed rule table as one pure function, so every row is pinned by a table test and the loop of card 12 only carries the actions out.
A new file `internal/loomcli/driverules.go` holds it.
No branch reads a row or recipe name or the reason text; each reads the envelope's fields, its `kind`, its `transient` class and facts the loop reads from disk.

`driveStop` holds what the loop envelope says: `OK`, `State`, `Reason`, `Kind`, `Transient`, `ParentNotice`, `Friction`, `EnvelopePath`, `Error`, the `loop` object's `Stop`, `CurrentProducer`, `HistoryLength`, `LoopState`, `InterruptPolicy`, `StatusMoved`, `TraceCopy` and `StderrPath`, the raw envelope bytes, and `FromStatus`.
`parseDriveStop` decodes the envelope `shedverbs.RunUntilStop` prints and refuses bytes that do not decode.
`stopFromStatus` builds the stop D12 acts on from a status alone, with `FromStatus` true, no kind and no reason.
`key` is the loop's `(current_producer, history_length, state)`, falling back to the top-level state, and equals `loomengine.StopKeyOf` of the status the stop left.

`driveFacts` carries the disk reads the loop makes before deciding:
- `EscalationPending`: the awaiting Bouncer row awaits its escalation, read through `arbiterDeps.PendingEscalation`;
- `VerifyMatches` and `VerifyTarget`: the Publish failure record matches the halt under `battenshed.RecordMatchesHalt`;
- `Watched`: a live batten watches the run;
- `VerifyRetrySpent`: the Go driver's settle records or batten's marker show the run's one retry spent;
- `BuildChanged`: the on-disk build differs from the last step's;
- `HandedOver`: a hand-over settle record exists for this stop key and build;
- `RepairCount`;
- `ResumeReturned`: an applied tackler `resume` for this stop key.
`repairCount` counts `restep` settle records of the stop key; `driveRepairCap` is the guide's two.
`binaryEligible` is the LLM driver's set (D3): `blocked`, a `producer`, `bootstrap`, `unseeded` or `interrupted` error, or a transient error, which reaches the driver only after its in-loop re-step.

`decideDrive` returns the first matching row in D3's order, as a `driveDecision` naming the action, the rule text for the stop report, `FrictionNote` and `WatchedVerifyLine`:
1. `done`: done.
2. a `busy` kind: busy exit.
3. an `ownership` kind, or an error with no kind: hand back.
4. `paused`: hand back, with `FrictionNote` when the stop came from a step and its reason is empty (D13).
5. `awaiting` with `EscalationPending`: the arbiter.
6. `awaiting` with a non-empty `ParentNotice`: relay and hand back.
7. any other `awaiting`: planned hand-back.
8. `blocked` with `VerifyMatches`:
   - watched: hand back with `WatchedVerifyLine` (D4);
   - unwatched with the retry unspent: verify goto to `VerifyTarget` then re-step;
   - unwatched with the retry spent: hand back.
9. a binary-eligible stop with `BuildChanged` and not `HandedOver`: hand over.
10. `ResumeReturned`: the arbiter.
11. a transient error: the tackler.
12. `interrupted` under `reinvoke` with `RepairCount` below the cap: re-step.
13. `interrupted` under `handback` or an empty policy: hand back.
14. a `producer`, `bootstrap` or `unseeded` error, a `reinvoke` stop at the cap, or any other `blocked` stop: the tackler.
Anything no row names hands back.

Tests in `driverules_test.go`:
- `TestDecideDrive`, a table with one row per D3 row, each naming the stop class in and the action out.
  It adds rows for a third `reinvoke` failure (`RepairCount` 2) going to the tackler, a watched verify halt carrying `WatchedVerifyLine`, an unwatched verify halt with the retry spent handing back, a non-matching record's `blocked` stop going to the tackler, a hand-over already made for this build going on to the next row, an empty-reason step `paused` stop carrying `FrictionNote`, and a status-derived `paused` stop carrying none.
- `TestParseDriveStop`, a table: a halted success envelope, an error envelope carrying `kind` and `transient`, an interrupted loop object with `status_moved` and its trace paths, and undecodable bytes refused.
`test-budget.yaml`'s `internal/loomcli` row rises by two.

**ImpactSummary:** new pure decisions in `internal/loomcli` with no caller until card 12; the budget row is the only existing file touched.
