---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/12-run-loop.md:30
    summary: A batch begin holds the state-mutation lease and then saves through saveRunState, which takes the same lease, so the begin deadlocks
    fix: >-
      In card 10, re-sign the created symbol as `plan:internal/weaverengine#saveRunState` -> `func saveRunState(geom Geometry, st *State, leaseHeld bool) error`,
      and state in its bullet that with `leaseHeld` set it skips the acquire and only loads, merges and writes, and with it unset it acquires `runplan.AcquireStateMutation` over the scratch directory first.
      In card 12, say that the begin sequence (the partition record, the batch record, `PlanWeftCommit`, the `StopRecord` clear) saves with `leaseHeld` true before it releases the lease,
      and that every other save of `Run`, `runBatch` and the resume passes false.
      Add a `TestRun` row in card 12 that begins a batch against a real state directory, so a second acquire of the lease would hang the test.
      The re-signed glyph passes `lym loom validate-plan --overlay` with no new blocking finding against the baseline.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/plan/13-run-reentry-and-resume.md:42
    summary: A re-entered last-rung attempt that ends done never clears StopRecord, so a later re-step re-enters a stop the run has moved past
    fix: >-
      In card 13, replace the clearing sentence with a rule that holds for the attempt path too.
      `StopRecord` is cleared, under `saveRunState`, at the moment any batch is recorded done by `endAttempt` or by the no-spawn path, and at the moment the verify re-run passes.
      Put the clear in card 10's `endAttempt` done branch and name it there, since `reenter` has already returned when a re-entered attempt ends.
      Add a `TestRun_Reentry` row where the stopped batch's last-rung attempt ends done and is the plan's last batch, the verify passes, the final record commit fails, and the next call begins no second re-entry.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lym/plan/13-run-reentry-and-resume.md:37
    summary: Re-entry's no-spawn check and its record-done step name no function, and card 10's checks sit inside endAttempt which needs an attempt and a report
    fix: >-
      In card 13, add to Create `plan:internal/weaverengine#checkStoppedBatch` -> `func checkStoppedBatch(deps RunDeps, st *State, plan *planparser.Plan, batch *runplan.BatchState) ([]string, error)`
      and `plan:internal/weaverengine#recordDoneWithoutSpawn` -> `func recordDoneWithoutSpawn(deps RunDeps, st *State, batch *runplan.BatchState) error`.
      The first runs the clean-tree check, the head check and `runplan.PostBatchChecks` with no renames over the batch's recorded start commit, and returns the failing reasons.
      The second marks the batch record terminal done with no digest, since `runplan.Distill` needs a report, sets `BatchEnds` and `LastEnd` from HEAD, saves through `saveRunState`, clears `StopRecord` (F2) and commits through `commitRecords`.
      Name the fields of `reentry` in the Intent (the batch to run, the rung to start on, and whether the verify stage is next), since `Run` branches on them.
      Card 10's `endAttempt` should call the same clean-tree and head checks through one shared unexported helper, so the two paths cannot drift.
      The added Create glyphs pass `lym loom validate-plan --overlay` with no new blocking finding against the baseline.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/plan/13-run-reentry-and-resume.md:26
    summary: resumeStrands returns only an error, so the rung and same-rung retry allowance of a batch that a crash left begun and not done are unspecified
    fix: >-
      In card 13, give `resumeStrands` the result type `plan:internal/weaverengine#resumeResult` -> `type resumeResult struct`, and re-sign it as `func resumeStrands(deps RunDeps, st *State) (resumeResult, error)`.
      State in the Intent what it carries for a resumed attempt, which `Run` hands to `runBatch`.
      A recordable attempt ends its batch done through `endAttempt`.
      A failed attempt starts the next attempt on the next rung.
      A lost attempt, including a not-found probe, retries once on its own rung when the batch has no earlier lost attempt on that rung, else it counts as failed.
      In card 12, say that a begin which finds a non-terminal batch record for its first fresh batch reuses that record's card set and start commit, and starts `runBatch` at the rung `resumeResult` or `reentry` names,
      and that a fresh partition whose first batch differs from the recorded one refuses naming `reentryWayForward`.
      Add `TestRun_Resume` rows for a crash after a failed attempt on rung 0 (the next attempt runs on rung 1 with the recorded start commit) and for a crash after a lost attempt (one same-rung retry).
      The glyph changes pass `lym loom validate-plan --overlay` with no new blocking finding against the baseline.
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/plan/12-run-loop.md:48
    summary: Card 12 writes outcome.yaml and card 23 reads it, but no card states its schema or declares a writer or reader both share
    fix: >-
      In card 12's Intent, state the file's content, which should be the keys `websterengine`'s `outcome.yaml` reads, `outcome` set to `done` and the other keys it requires, so the two engines' files read alike.
      In card 23's Intent, say `weaverengine.RunInFlight` decodes exactly those keys and treats any other `outcome` value or an undecodable file as the error its `TestRunInFlight` row names.
      No target changes.
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/plan/18-weaver-implementer-row.md:42
    summary: Card 18 makes weaverEntry require Env.WeaverRun but lists no fill of it in loomCLI.wire, while card 16 says the adapter defaults to weaverengine.Run
    fix: >-
      In card 18's wire bullet, add that `loomCLI.wire` sets `WeaverRun` to `weaverengine.Run` explicitly, as `WebsterRun` is set.
      In card 16, drop the words default `weaverengine.Run` from the `WeaverProducer.Call` bullet, so the adapter calls only the injected `run`.
      This keeps `weaverengine.Run` reachable from `cmd/lym` through the wiring alone (PATTERN-test-only-code).
      No target changes.
  - id: F7
    severity: LOW
    class: consistency
    location: _lym/plan/13-run-reentry-and-resume.md:49
    summary: The last TestRun_Resume row repeats the new-attempt-number assertions that TestRun and TestRun_Reentry already hold
    fix: >-
      In card 13, delete the row `a retry and a re-entry attempt each take a new attempt number` from `TestRun_Resume`.
      Card 12's `TestRun` row on a machinery-loss retry holds the retry's new attempt number,
      and `TestRun_Reentry`'s row on a lost re-entry attempt holds the re-entry attempt's.
---

### [BLOCKING] A batch begin deadlocks on the state-mutation lease

**Location:** `_lym/plan/12-run-loop.md:30`, `_lym/plan/12-run-loop.md:36`, `_lym/plan/10-batch-record.md:35`
**Issue:** Card 12 has each batch begin "under the state-mutation lease" and records the partition, the batch record, `PlanWeftCommit` and the `StopRecord` clear inside it.
The same card sends "every state save of `Run`, `runBatch` and the resume" through `saveRunState`, and card 10 has `saveRunState` take that lease itself before it loads and writes.
`runplan.AcquireStateMutation` calls `lock.AcquireWriteLock`, which opens a new `flock` on `mutate.lock` and blocks until it is free.
A second acquire in the same process opens a second file description on the same file and never returns.
So the begin's own save blocks on the lease the begin holds.
`websterengine`'s state phase avoids this by holding the lease and calling its lease-free `SaveState`.
The plan has no such split, so `TestRun` over a real scratch directory hangs at the first begin.
Round 1's fix for its F7 introduced `saveRunState` and routed every save through it; that fix is the source, and round 1's fixer report disputed nothing here.

### [MEDIUM] StopRecord outlives a re-entry that ends through the attempt path

**Location:** `_lym/plan/13-run-reentry-and-resume.md:42`
**Issue:** `reenter` clears `StopRecord` only when the stopped batch is recorded done with no spawn or the verify re-run passes.
Every other re-entry returns a `reentry` to `Run`, which then runs the last-rung or needs-parent attempt, and `reenter` has already returned when that attempt ends done.
Card 12 clears the record only at a later batch begin.
When the re-entered batch is the plan's last, no begin follows, and a fault Stuck, which records no `StopRecord` (card 5), leaves the old batch stop standing.
Example: the last-rung attempt ends done, the verify passes, and the final record commit fails, so `Run` returns Stuck with the re-step as its way forward.
The next call finds `StopRecord` still set, so `reenter` runs the post-batch checks again on a batch already done, a stop the run has moved past.
This contradicts card 12's own statement that a re-step never re-enters such a stop, and the decision record's D7 re-entry that judges only a batch that is still stopped.
Round 1's F5 fix added the clearing in `reenter` and at begin; this finding is the case those two sites leave open, not a regression of what that fix covers.

### [MEDIUM] Re-entry's no-spawn path has no function of its own

**Location:** `_lym/plan/13-run-reentry-and-resume.md:37`
**Issue:** Card 13 says `reenter` "runs the post-batch checks (card 10) on the stopped batch against its recorded start commit" and records the batch done with no spawn.
Card 10 holds the clean-tree check, the `head_sha` check and the `runplan.PostBatchChecks` call only inside `endAttempt`, whose input is an attempt, its verdict and its parsed report.
A stopped batch re-entered with no spawn has no new attempt and no report, and `runplan.Distill` takes a `*Report`.
Neither the check sequence nor the record-done step (done record, `BatchEnds`, `LastEnd`, save, commit) is a target of any card, so a function `Run` reaches through `reenter` has no card.
`reentry` is also declared with no stated fields, so the contract between `reenter` and `Run` is open.
The rubric's checks do not cover this, and the decision's D7 (full post-batch checks, then done with no spawn) rests on it.

### [MEDIUM] The continuation after a resumed or crashed attempt is unspecified

**Location:** `_lym/plan/13-run-reentry-and-resume.md:26`
**Issue:** `resumeStrands` returns only an error.
Its attempts end by `endAttempt` as in a normal run, so a resumed attempt can end done, failed or lost, but nothing tells `Run` what comes next.
Card 12's begin "records the batch's card set, card hashes and start commit once" and `runBatch` takes a `startRung`, yet no text says how `Run` finds a begun, non-terminal batch after a crash, reuses its record, picks `startRung` from its last attempt, or counts the same-rung retry already spent.
D3 and D5 make the card set and start commit fixed for every attempt of a batch and give one same-rung retry per machinery loss, so a crash must not reset either.
D12 says a not-found probe "counts as a machinery loss", which needs a retry rule at this point.
Without it an implementer must invent the rule, and `TestRun_Resume` has no row for the next attempt's rung.

### [LOW] outcome.yaml has no stated schema

**Location:** `_lym/plan/12-run-loop.md:48`
**Issue:** Card 12 writes `outcome.yaml` done and card 23's `RunInFlight` reads it, with an "unreadable outcome" row.
`websterengine`'s reader is unexported and strict about its keys, so Weaver's engine cannot call it.
No card says which keys the file holds, so the writer in `Run` and the reader in `RunInFlight` can disagree.

### [LOW] WeaverRun has no fill in loom's wiring

**Location:** `_lym/plan/18-weaver-implementer-row.md:42`
**Issue:** Card 18 lists `WeaverRun` among the inputs `weaverEntry` requires.
Its `loomCLI.wire` bullet fills `WeaverDeps` and `ImplementerSeam` but not `WeaverRun`, and `nilLegal` does not list it.
Card 16 says the adapter defaults to `weaverengine.Run`.
`loomcli/wiring.go` fills `WebsterRun` explicitly because the Webster entry fails on a nil.
As written, either `TestWire_EverySeamFilled` fails or `weaverengine.Run` is reached only through a default no card wires, which `PATTERN-test-only-code` counts as unreachable from a `main` package.

### [LOW] A resume test row repeats rows held elsewhere

**Location:** `_lym/plan/13-run-reentry-and-resume.md:49`
**Issue:** `TestRun_Resume` ends with "a retry and a re-entry attempt each take a new attempt number".
Card 12's `TestRun` already has "a machinery loss ... retries once on the same rung with a new attempt number",
and card 13's own `TestRun_Reentry` has "a lost re-entry attempt retries once on the last rung with a new attempt number".
Both assert the same behavior at the same surface (`PATTERN-test-speed`, Test economy).

## Focus verdicts

- Cards 12 and 13, functions called from `Run`: the targets of cards 7 to 11 cover the batch strand, record, verify and fixer units, and card 12's Uses cover the shared runplan calls.
  The calls with no declared target are the re-entry check and record-done step, the resume continuation and the `outcome.yaml` write (F3, F4, F5).
  Each prescribed refusal and Stuck branch of cards 12 and 13 has a named `TestRun`, `TestRun_Reentry` or `TestRun_Resume` row, so no finding there.
- Card 13 re-entry gates: the card and batch gates run at HEAD before a no-spawn done, and a row with `gates` entries never takes that path, as D7 says.
  The one gap is the stop record's clearing (F2).
- Card 2 and `weaver.yaml`: `verify_gate` is `CategoryGuard`, so the local layer cannot weaken the gate behind Done.
  `ladder` and `fixer_timeout_min` are `CategoryNone`; a changed ladder only changes which models run, and an expired fixer timeout leads to a verify re-run and not to a pass, so neither hollows a guard. Not a defect.
- Cards 10, 11 and 21, state-file overwrite: `saveRunState` merges `Delivered` under the lease, and `RecordDelivery` writes under the same lease, so a delivery survives a save.
  The merge is defeated only by the lease conflict in F1.
- Card 14, reset: each commit or archive branch states its bound (the marker, the own-paths rule, the lock and probe refusals), and `TestReset` has a row for each refusal. Not a defect.
- Cards 9, 10 and 11 test through `TestRun` and add no per-helper test. Card 3's `TestPaths` and `TestWeaverGeometry` cover distinct things. The only redundancy found is F7.
- `PATTERN-test-only-code`: every production function the cards add is reached from a `main` package through the weaver CLI, the loom wiring or the shed adapter, except the `weaverengine.Run` wiring question in F6.
